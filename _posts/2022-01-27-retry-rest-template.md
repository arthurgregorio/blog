---
title: 'Integrando retry template com rest template no Spring em Kotlin'
date: 2022-01-27 22:00:00 +0300
categories: [Programação]
tags: [programacao, kotlin, spring]
image:
  path: '/v1643331650/blog/retry-rest-template/Spring-Framework_nuewfg.png'
  width: 1000
  height: 400
---

É impossível negar que o *RestTemplate* ainda é muito utilizado no universo do spring, logo, ter resiliência com ele
é algo necessário, mas como podemos fazer isso em Kotlin integrando com o *RetryTemplate*?

## Um pouco de história

Antes de seguir, precisamos esclarecer uma coisa: se seu projeto é novo e ainda não usa o *RestTemplate* mas você esta pensando
em colocá-lo, pare e não faça isso! O motivo é muito simples e pode ser encontrado nas documentações do spring.

Retirado [daqui](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html):

> NOTE: As of 5.0 this class is in maintenance mode, with only minor requests for changes and bugs to be accepted going forward. Please, consider using the org.springframework.web.reactive.client.WebClient which has a more modern API and supports sync, async, and streaming scenarios.

Ou seja, este post se aplica para clientes onde:

- A versão utilizada do spring ainda não oferece suporte ao WebClient
- O projeto já utiliza o RestTemplate e não há previsão de mudar para algo mais "atual"
- Você quer fazer algo com RestTemplate mesmo sabendo que ele esta em modo de manutenção

Isto posto, o motivo de eu estar escrevendo sobre algo "em modo manutenção" esta no início do post, RestTemplate ainda é muito utilizado e eu precisei
implementar o retry nele, achei justo falar sobre isso por aqui pq achei poucos conteúdos sobre o tema.

Depois pretendo escrever algo semelhante para o WebClient também, então fique ligado!

## Resiliência em clientes HTTP

Eu não vou escrever exatamente sobre isso pois há muito conteúdo bom sobre o tema na internet, mas para facilitar e justificar o motivo de precisarmos adicionar
estratégias a fim de garantir a resiliência de nossas chamadas HTTP, vou deixar essa talk completíssima do [Rafael Ponte](https://www.linkedin.com/in/rponte/) sobre o tema, nela todos os pontos que vamos abordar aqui são devidamente explicados e exemplificados com cenários de uso.

{% include youtube.html id="TMmN9cR_IsM" %}

## Configurando

> O Código fonte completo desse artigo você pode encontrar [aqui](https://github.com/arthurgregorio/exemplos-blog/tree/main/rest-retry-template)

Primeiro vamos precisar configurar o [Spring Retry](https://docs.spring.io/spring-batch/docs/current/reference/html/retry.html), que inicialmente era uma parte do Spring Batch mas foi transformado em um novo projeto a parte e pode ser utilizado em qualquer cenário onde você precisa que um determinado método (ou trecho de código) seja reexecutado dada uma determinada condição de erro (exception).

```kotlin
@EnableRetry
@Configuration
class CommonsConfiguration {

    @Bean
    fun retryTemplate(): RetryTemplate {
        return RetryTemplate.builder()
            .maxAttempts(3)
            .exponentialBackoff(50, 2.0, 3000, true)
            .retryOn(HttpServerErrorException::class.java)
            .build()
    }

    @Bean
    fun restTemplate(builder: RestTemplateBuilder): RestTemplate {
        return builder.build()
    }
}
```

> O RetryTemplate tem mais opções para configuração e elas podem ser vistas [aqui](https://github.com/spring-projects/spring-retry).

Dada a configuração, temos:

Na linha #1 a anotação que vai ativar o uso do RetryTemplate via annotations (não é obrigatório pois não vou mostrar isso aqui). Nas linhas seguintes temos as configurações para que o retry possa fazer um backoff exponencial e que ele só funcione em erros de tipo http-5xx, afinal erros do tipo http-4xx não são possíveis de retry visto que representam (ou deveriam representar!) um estado inconsistente dos dados que foram enviados ao servidor [1], por isso são chamados de *client errors* e por fim, o builder do nosso RestTemplate.

> [1] Existem dois códigos http da familia 4xx que podem sim sofrer retry, seriam o 408 e o 429 ([obrigado Rafael Ponte!](https://twitter.com/gregorioarthur/status/1487267330111549445)) porém, no caso específico do 429 pode ser que você precise de alguma lógica de negócio antes visto que ele pode indicar que você atingiu o limite de requests por segundo/minuto/hora da API de destino

## Cliente tolerante a falhas

Agora que configuramos o que precisa ser usado, vamos criar um componente para ser nosso cliente http tunado (tolerante a falhas), vou chamá-lo de ```EnhancedHttpClient```:

```kotlin
@Component
class EnhancedHttpClient(
    private val restTemplate: RestTemplate,
    private val retryTemplate: RetryTemplate
) {

    fun <T : Any> exchangeWithRetry(requestEntity: RequestEntity<*>, responseType: KClass<T>): ResponseEntity<T> {
        return retryTemplate.execute<ResponseEntity<T>, RestClientException> {
            restTemplate.exchange(
                requestEntity.url.toString(),
                requestEntity.method!!,
                requestEntity,
                responseType.java
            )
        }
    }
}
```

O código parece meio complexo mas ele nada mais é do que uma chamada aninhada do RestTemplate dentro de um RetryTemplate, assim, em caso de erros (conforme nossa configuração) o retry irá executar a quantidade de tentativas configuradas dentro do período de backoff especificado.

A complexidade maior fica por conta que dei uma "generificada" nele a fim de possibilitar o reuso por diversos clientes, isso ajuda na manutenção depois!

## Testando

Agora que temos tudo configurado e pronto, podemos testar! Vamos iniciar criando uma service que irá usar nosso cliente fazendo uma chamada para uma API rest externa
e que pode, eventualmente, nos retornar algum erro:

```kotlin
@Service
class CatService(
    @Value("\${cat-service.url}")
    private val catServiceUrl: String,
    private val enhancedHttpClient: EnhancedHttpClient
) {

    fun findCatFact(): String {

        // O cliente http ainda poderia estar encapsulado em outro componente, isso seria util para termos
        // um maior reaproveitamento do código

        val uri = UriComponentsBuilder.fromHttpUrl(catServiceUrl).build().toUri()
        return enhancedHttpClient.exchangeWithRetry(RequestEntity("", HttpMethod.GET, uri), String::class).body!!
    }
}
```

Vamos escrever alguns testes:

```kotlin
@SpringBootTest
class ApplicationTests {

    @Value("\${cat-service.url}")
    private lateinit var catServiceUrl: String

    @Autowired
    private lateinit var restTemplate: RestTemplate

    @Autowired
    private lateinit var catService: CatService

    private lateinit var mockServer: MockRestServiceServer

    @BeforeEach
    fun setup() {
        mockServer = MockRestServiceServer.createServer(restTemplate)
    }

    @Test
    fun `should find a cat fact`() {

        mockServer.expect(once(), requestTo(catServiceUrl))
            .andExpect(method(HttpMethod.GET))
            .andRespond(
                withStatus(HttpStatus.OK)
                    .contentType(MediaType.APPLICATION_JSON)
                    .body("Some random cat fact")
            )

        val catFact = catService.findCatFact()
        assertThat(catFact).isEqualTo("Some random cat fact")

        mockServer.verify()
    }

    @Test
    fun `should retry three times before throw error`() {

        mockServer.expect(times(3), requestTo(catServiceUrl))
            .andExpect(method(HttpMethod.GET))
            .andRespond(withStatus(HttpStatus.INTERNAL_SERVER_ERROR))

        assertThrows<HttpServerErrorException> { catService.findCatFact() }

        mockServer.verify()
    }

    @Test
    fun `should not retry when client error occur`() {

        mockServer.expect(once(), requestTo(catServiceUrl))
            .andExpect(method(HttpMethod.GET))
            .andRespond(withStatus(HttpStatus.BAD_REQUEST))

        assertThrows<HttpClientErrorException> { catService.findCatFact() }

        mockServer.verify()
    }
}
```

A ideia por trás dos testes é muito simples: realizamos uma chamada http para o servidor que esta mockado e quando for um caso de erro do servidor, o retry deve atuar
e fazer com que o método seja chamado 3x antes de retornar a exception para a classe que chamou o método. No caso de erro do cliente, o retry não deve atuar.

Ficou com dúvidas? Comenta ai em baixo que vou respondendo!
