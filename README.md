# Exemplo de Webhook com Spring Boot

Este é um exemplo simples de implementação de um webhook utilizando Spring Boot. O serviço é responsável por enviar pedidos finalizados para uma URL específica via webhook.

## Configuração

Antes de começar, certifique-se de ter as seguintes dependências em seu projeto:

- [Spring Boot](https://spring.io/projects/spring-boot)
- [Lombok](https://projectlombok.org/)
- [RestTemplate](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)

## Classe WebhookService

A classe `WebhookService` é responsável por enviar pedidos finalizados via webhook. Aqui está uma explicação básica do código:

```java
@Service
@Slf4j
@RequiredArgsConstructor(onConstructor = @__(@Autowired))
public class WebhookService {

    // Injeção de dependências
    private final RestTemplate restTemplate;
    private final OrderRepository orderRepository;
    private final UserRepository userRepository;
    private final AwardOrderRepository awardOrderRepository;

    // ...

    @Scheduled(fixedRate = 1200000)
    public void sendOrder() {

        log.info("Iniciado o webhook de envio de pedidos.");

        // Lógica para enviar pedidos finalizados
        // ...

        String payload = "Pedido com ID " + order.getId() + " finalizado";

        sendOrderWebhookWithRetries(orderModel, user.getOrderWebhookUrl(), payload, 3);
    }

    public void sendOrderWebhookWithRetries(OrderModel orderModel, String webhookUrl, String payload, int maxRetries) {
        // Lógica para enviar webhook com tentativas de retentativas
        // ...

      int currentRetry = 0;

        while (currentRetry < maxRetries) {
            try {
                sendWebhook(webhookUrl, payload);

                orderModel.setStatus(OrderStatusEnum.FINALIZADO.name());
                orderModel.setSendWebhook(payload);
                orderRepository.save(orderModel);
                return;
            } catch (Exception e) {
                currentRetry++;
                log.error("Erro no envio via Webhook (Tentativa " + currentRetry + "): " + e.getMessage());

                if (currentRetry < maxRetries) {
                    try {
                        Thread.sleep(RETRY_DELAY_SECONDS);
                    } catch (InterruptedException ex) {
                        Thread.currentThread().interrupt();
                    }
                } else {
                    String webhookFailureMessage = "Não foi possível enviar o webhook após " + maxRetries + " tentativas";
                    log.error(webhookFailureMessage);

                    orderModel.setStatus(OrderStatusEnum.ERRO_NO_EVIO.name());
                    orderModel.setSendWebhook(webhookFailureMessage);
                    orderRepository.save(orderModel);
                }
            }
        }
    }

    private void sendWebhook(String webhookUrl, String payload) {
        // Método para enviar o webhook
        // ...

        restTemplate.setInterceptors(Collections.singletonList(new CustomUserAgentInterceptor("TesteWebhookAgent/1.0")));
        restTemplate.postForObject(webhookUrl, payload, String.class);
        log.info(payload);
    }
}
```

# Classe CustomUserAgentInterceptor

A classe `CustomUserAgentInterceptor` é uma implementação de `ClientHttpRequestInterceptor` utilizada para adicionar um cabeçalho de agente do usuário personalizado a uma solicitação HTTP.

```java
public class CustomUserAgentInterceptor implements ClientHttpRequestInterceptor {
    private final String userAgent;

    public CustomUserAgentInterceptor(String userAgent) {
        this.userAgent = userAgent;
    }

    @Override
    public ClientHttpResponse intercept(HttpRequest request, byte[] body, ClientHttpRequestExecution execution) throws IOException {
        request.getHeaders().set("User-Agent", userAgent);
        return execution.execute(request, body);
    }
}
```


## Configuração do Schedule

O método `sendOrder` é agendado para ser executado a cada 1200000 milissegundos (20 minutos) e verifica pedidos pendentes para envio via webhook.

## Envio do Webhook com Retentativas

O método `sendOrderWebhookWithRetries` envia o webhook com tentativas de retentativas em caso de falha.

## Envio Real do Webhook

O método `sendWebhook` utiliza o `RestTemplate` para enviar o payload para a URL do webhook.

Esta é uma visão geral simples do código. Certifique-se de configurar adequadamente as dependências e personalizar conforme necessário para o seu caso de uso.

Espero que isso ajude! Se precisar de mais informações ou esclarecimentos, estou à disposição.

