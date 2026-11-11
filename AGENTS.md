# AGENTS.md

Guia portável para agentes de IA que trabalham neste repositório.

## O que é este repositório

Suíte de teste de carga e resiliência em k6 contra o OWASP Juice Shop, que sobe local via
Docker. Quatro cenários: smoke, load, stress e spike.

## Subir e derrubar o alvo

```bash
docker compose up -d     # http://localhost:3000
docker compose down
```

## Rodar os cenários

```bash
k6 run smoke.js      # sempre primeiro, valida alvo e script
k6 run load.js
k6 run stress.js
k6 run spike.js
```

## Convenções

- Cada script aceita `-e BASE_URL=...` e tem como default `http://localhost:3000`.
- Cada script declara `thresholds` que definem passou ou falhou, pra que `k6 run` saia com
  código de erro quando o SLO é violado. Útil num pipeline.
- `smoke` roda antes dos demais; se ele falha, não faz sentido medir carga maior.
- Alvo sempre local ou de treino autorizado, nunca sistema de terceiros.

## Idioma

Documentação e comentários em português. Termos técnicos e identificadores permanecem na
forma original.
