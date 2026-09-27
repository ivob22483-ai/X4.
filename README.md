# X4Payment — Backend

Backend real (não é uma demonstração): base de dados, autenticação,
cadastro/aprovação de comerciantes, produtos, checkout público e ligação
preparada para M-Pesa e e-Mola.

## 1. Correr localmente

```bash
npm install
cp .env.example .env
# edita o .env: define JWT_SECRET e ADMIN_TOKEN com valores fortes e aleatórios
npm start
```

O servidor sobe em `http://localhost:4000`. Testa com:
```bash
curl http://localhost:4000/api/health
```

## 2. Principais rotas

| Rota | Quem usa | Descrição |
|---|---|---|
| `POST /api/auth/register` | comerciante | cria conta (multipart, campo `documento` obrigatório) |
| `POST /api/auth/login` | comerciante | devolve token JWT |
| `GET /api/merchants/me` | comerciante | perfil e estado da conta |
| `GET /api/merchants/me/balance` | comerciante | saldo pendente / disponível |
| `POST /api/merchants/me/withdrawals` | comerciante | solicita levantamento do saldo disponível |
| `POST /api/products` | comerciante | cadastra produto (só se conta aprovada) |
| `GET /api/checkout/:slug` | público | dados do checkout |
| `POST /api/checkout/:slug/pay` | público | inicia o pagamento |
| `POST /api/payments/webhook/mpesa` `/emola` | operador | confirmação do pagamento |
| `GET/POST /api/admin/...` | tua equipa | aprovação de contas, produtos, saldo, levantamentos (header `x-admin-token`) |

## 3. O que já está pronto

- Conta com password encriptada (bcrypt) e sessão por JWT.
- Upload do documento de identificação e fluxo de aprovação/rejeição.
- Produtos com tipo (e-book, curso, vídeo, link, serviço, outro) e aprovação.
- Checkout público com geração automática de link por produto.
- Registo de transações, saldo pendente → disponível (com prazo de retenção configurável) → levantamento com taxa.
- Painel administrativo via API para aprovar contas, produtos, libertar saldo e aprovar levantamentos.

## 4. O que falta — e não é código, é negociação/burocracia

Isto é o que **tu** precisas de tratar diretamente, antes disto poder processar dinheiro a sério:

1. **Acordo com a Vodacom M-Pesa** — candidatar-te como agregador/comerciante API, obter API key, chave pública e "service provider code". Depois, substituis o conteúdo de `src/providers/mpesa.js` pela chamada real.
2. **Acordo com a Movitel e-Mola** — o mesmo processo, credenciais próprias, ficheiro `src/providers/emola.js`.
3. **Conta bancária empresarial** — para onde os operadores liquidam o dinheiro recebido, e de onde saem os levantamentos para os comerciantes.
4. **Enquadramento regulatório** — em Moçambique, prestação de serviços de pagamento é regulada pelo Banco de Moçambique; confirma com um advogado/contabilista se precisas de licença ou registo como instituição de moeda eletrónica/agregador de pagamentos.
5. **Domínio e hospedagem** — comprar um domínio (ex. `x4payment.co.mz`) e escolher onde correr isto (Railway, Render, VPS, etc.), com HTTPS.
6. **KYC mais robusto** — hoje o documento fica só guardado em disco (`uploads/`); antes de produção, considera cifrar esses ficheiros e/ou usar um serviço de verificação de identidade.
7. **Admin real** — troca o `ADMIN_TOKEN` fixo por contas de administrador com login próprio e registo de auditoria (quem aprovou o quê e quando).

## 5. Notas de segurança antes de ires ao ar

- Nunca uses os valores de exemplo do `.env` — gera segredos novos.
- Ativa HTTPS em produção (a maioria dos serviços de hospedagem trata disto automaticamente).
- Considera limitar tentativas de login (rate limiting) para evitar ataques de força bruta.
- Faz backups regulares do ficheiro da base de dados (`data/x4payment.db`).
