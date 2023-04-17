---
title: 'Queries mais dinâmicas com spring data JPA'
date: 2023-04-13 22:00:00 +0300
categories: [Programação]
tags: [programacao, kotlin, spring, jpa]
image:
  path: '/v1681691727/blog/spring-data-specification/spring-data-spec.001_wazmup.jpg'
  width: 1000
  height: 400
---

Por mais que você pense que sabe bastante sobre Spring, eu lhe digo: _sempre há algo novo a se aprender_.

Hoje vamos falar um pouco da API de especification presente no módulo do spring data, com ela podemos fazer muitas coisas
interessantes, como, por exemplo, queries bem dinâmicas!

> Para introduzir bem o assunto, vou dar um contexto me baseando no webBudget e com alguns casos de uso por lá, ah! e se
> você ainad não conhece o projeto, clica aqui e confere!

## O problema

Observem o exemplo abaixo, nele vemos um método simples utilizando a anotação `@Query` para definir uma pesquisa no banco
e retornar um terminado resultado, no caso uma lista de centros de custo que deem match com o filtro recebido como parâmetro.

```kotlin
@Repository
interface CostCenterRepository : DefaultRepository<CostCenter> {

    @Query("from CostCenter c " +
            "where (:#{#filter} is null or c.name like :#{#filter}) " +
            "and (:#{#filter} is null or c.description like :#{#filter}) ")
    fun findByFilter(filter: String?): List<CostCenter>
}
```

Nada muito complicado até aqui, exceto pelo fato que essa pesquisa passa a ficar mais complicada se precisarmos adicionar
paginação ou até mesmo aumentar a quantidade de filtros, isso sem falar no fato que "quase" temos um `If` dentro da query
para garantir, por exemplo, que se o valor recebido for nulo, todos os registros devem ser retornados.

Dado o problema, em algumas pesquisas na vasta documentação do spring alguns artigos escritos pelo pessoal que mantem o
projeto, encontrei [esse link](https://spring.io/blog/2011/04/26/advanced-spring-data-jpa-specifications-and-querydsl).

## A primeira solução

Para iniciarmos, nosso repositório passou a extender mais uma _interface_, agora a `JpaSpecificationExecutor` faz parte
de nosso `DefaultRepository` (base para os demais), você pode vê-lo [aqui](https://github.com/web-budget/back-end/blob/v4.0.0-alpha.1/src/main/kotlin/br/com/webbudget/infrastructure/repository/DefaultRepository.kt).

Feita a config básica, a implementação mostrada anteriormente passou a funcionar tal como descrevo abaixo:

Uma `data class` que representa o filtro foi criada para englobar a "montagem" da query via specification, você pode ver
isso sendo feito abaixo:

```kotlin
data class CostCenterFilter(
    val filter: String?,
    val status: StatusFilter?
) : SpecificationSupport<CostCenter> {

    override fun buildPredicates(
        root: Root<CostCenter>,
        query: CriteriaQuery<*>,
        builder: CriteriaBuilder
    ): List<Predicate> {

        val predicates = mutableListOf<Predicate>()

        if (!filter.isNullOrBlank()) {
            predicates.add(
                builder.or(
                    builder.like(builder.lower(root.get("name")), likeIgnoringCase(filter)),
                    builder.like(builder.lower(root.get("description")), likeIgnoringCase(filter))
                )
            )
        }

        if (status != null && status != StatusFilter.ALL) {
            predicates.add(builder.equal(root.get<Boolean>("active"), status.value))
        }

        return predicates
    }
}
```

A classe inteira você encontra [aqui](https://github.com/web-budget/back-end/blob/v4.0.0-alpha.1/src/main/kotlin/br/com/webbudget/application/payloads/registration/CostCenterPayloads.kt#L29-L57).

Depois disso, a controller que recebe o filtro como parâmetro no método de busca, chama o método `toSpecification` definido
na interface `SpecificationSupport`. Este método vai chamar o método acima para contruir os predicados da nossa especification
e fim!

Ao chamar o método `findAll` default do spring data, você apenas terá que passar os parâmetros corretos, no caso mostrado [aqui](https://github.com/web-budget/back-end/blob/v4.0.0-alpha.1/src/main/kotlin/br/com/webbudget/application/controllers/registration/CostCenterController.kt#L36),
invocamos aquele que recebe uma specification e um pageable como parâmetros.

Isso iria garantir que caso precise adicionar mais parâmetros a minha busca, bastaria adicionar um novo predicate na lista
de predicates e pronto, spring cuidaria do resto. Más, nem tudo são flores, detalhes sobre como criar uma query via
especification agora estavam espalhados por uma camada muito externa da minha aplicação, e isso não me deixou feliz.

## Refatorando

Voltei as pesquisas e depois de ler alguns artigos (referências no final deste) entendi que estes detalhes de como criar
a especificação poderia estar até mesmo dentro do meu repositório, sendo assim, chegamos a esta nova implementação para os
centros de custo:

```kotlin
@Repository
interface CostCenterRepository : DefaultRepository<CostCenter> {

    // other methods omitted

    object Specifications : SpecificationHelpers {

        fun byName(name: String?) = Specification<CostCenter> { root, _, builder ->
            name?.let { builder.like(builder.lower(root.get("name")), likeIgnoringCase(name)) }
        }

        fun byDescription(description: String?) = Specification<CostCenter> { root, _, builder ->
            description?.let { builder.like(builder.lower(root.get("description")), likeIgnoringCase(description)) }
        }

        fun byActive(active: Boolean?) = Specification<CostCenter> { root, _, builder ->
            active?.let { builder.equal(root.get<Boolean>("active"), active) }
        }
    }
}
```

Desta forma, para compor um novo filtro, criámos um método aqui que defina a especificação e então poderemos escrever
da seguinte maneira em nosso filtro:

```kotlin
data class CostCenterFilter(
    val filter: String?,
    val status: StatusFilter?
) : SpecificationSupport<CostCenter> {

    override fun toSpecification(): Specification<CostCenter> {
        return byActive(status?.value).and(byName(filter).or(byDescription(filter)))
    }
}
```

Muito mais simples né? Não perdemos nenhuma das capacidades anteriores e ainda ganhamos algumas novas:

1. Os predicados podem ser reutilizados em outras áreas do projeto
2. O filtro apenas monta com base nos métodos, nenhum outro detalhe interno do framework foi exposto
3. Muito mais fácil de entender e manter

## Referências

- [https://vladmihalcea.com/spring-data-jpa-specification/](https://vladmihalcea.com/spring-data-jpa-specification/)
- [https://reflectoring.io/spring-data-specifications/](https://reflectoring.io/spring-data-specifications/)

