# k6 Performance Testing

Suíte de teste de carga e resiliência escrita em [k6](https://grafana.com/docs/k6/latest/).
A ideia é medir como uma aplicação web se comporta sob pressão: quanto demora pra responder,
quantas requisições aguenta por segundo e a partir de onde começa a sofrer. O alvo é o OWASP
Juice Shop, que sobe local via Docker, então os testes rodam contra algo real sem depender
de ambiente externo.

## Subir o alvo

```bash
docker compose up -d        # alvo em http://localhost:3000
docker compose down         # derruba quando terminar
```

## Os cenários

Cada script responde uma pergunta diferente. Rode o smoke primeiro: se ele falha, não faz
sentido medir carga maior.

| Script | Pergunta que responde | Duração |
|--------|-----------------------|---------|
| `smoke.js` | O alvo e o script estão sãos? (1 usuário) | 15s |
| `load.js` | Como fica a latência no tráfego normal? (30 usuários) | ~70s |
| `stress.js` | Onde está o limite? Rampa até 200 usuários | ~70s |
| `spike.js` | Aguenta um pico súbito e se recupera depois? | ~55s |

## Rodar

```bash
k6 run smoke.js
k6 run load.js
k6 run stress.js
k6 run spike.js
```

Pra apontar pra outro host, passe a variável de ambiente:

```bash
k6 run -e BASE_URL=http://localhost:3000 load.js
```

## Como ler os resultados

- **`http_req_duration` com `p(95)`** é a latência que importa: 95% das respostas ficaram
  abaixo desse valor. A média engana, a p95 mostra a cauda que o usuário sente.
- **`http_req_failed`** é a taxa de respostas que não foram 2xx.
- **Thresholds** são os limites que definem passou ou falhou. Cada script declara os seus,
  então o `k6 run` sai com código de erro quando o SLO é violado, pronto pra um pipeline.

## O que eu medi neste alvo

- **Load, 30 usuários:** p95 de 38ms, 0% de erro. Tranquilo no dia a dia.
- **Stress, 200 usuários:** p95 subiu pra 1,72s, mas 0% de erro. A latência piorou 45 vezes
  e o sistema degradou sem quebrar.
- **Spike, pico na home:** 7406 req/s com p95 de 25ms. A home é estática e quase não sente.

O contraste conta uma história: a home estática aguenta o pico numa boa, mas a busca, que é
um endpoint dinâmico, foi o que degradou no stress. O gargalo mora no que processa, não no
que só serve arquivo.

## Escopo

Tudo roda contra um alvo local, feito de propósito pra treino. É um laboratório de
performance, não um teste contra sistema de terceiros.
