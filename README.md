# Contas Fixas

Site para controle de despesas fixas mensais, com cálculo automático de totais e dashboard comparativo entre meses.

🔗 **Site:** [www.contasfixas.com.br](https://www.contasfixas.com.br)

---

## 📋 Sobre o projeto

O Contas Fixas permite que qualquer pessoa cadastre suas despesas fixas mensais (aluguel, condomínio, água, energia, internet, mercado etc.), acompanhe o total gasto a cada mês e compare a evolução entre meses diferentes — tudo isso sem precisar de planilhas.

O acesso é liberado após um pagamento único via Pix (R$35), processado em tempo real através da integração com o Mercado Pago.

## ✨ Funcionalidades

- **Conta e autenticação**
  - Cadastro com confirmação por e-mail
  - Login / logout
  - Recuperação de senha ("Esqueci minha senha")
  - Troca de senha autenticado
  - Exclusão de conta completa (remove login, dados e despesas)
  - Botão de mostrar/ocultar senha em todos os campos
- **Despesas**
  - 10 despesas fixas padrão, já cadastradas na ativação da conta
  - Possibilidade de adicionar ou remover despesas livremente
  - Edição mês a mês, com total calculado automaticamente
  - Valores no padrão brasileiro (ponto para milhar, vírgula para decimal)
- **Dashboard comparativo**
  - Comparação entre dois meses escolhidos livremente
  - Variação em R$ e em % por categoria e no total
  - Gráfico com a evolução dos meses do ano
- **Pagamento**
  - Cobrança Pix gerada dinamicamente (QR Code + código copia-e-cola)
  - Confirmação automática do pagamento via webhook (sem ação manual)
- **Perfil**
  - Foto, telefone, endereço e informações adicionais
- **Páginas institucionais**
  - Termos de Uso
  - Política de Privacidade (LGPD)

## 🧱 Arquitetura

Este repositório contém **apenas o front-end** (hospedado como site estático no GitHub Pages). Toda a lógica de servidor roda em serviços externos:

| Camada | Serviço usado |
|---|---|
| Hospedagem do site | GitHub Pages |
| Domínio | registro.br |
| Banco de dados + autenticação | [Supabase](https://supabase.com) (Postgres) |
| Envio de e-mails (confirmação, redefinição de senha) | [Resend](https://resend.com) |
| Pagamento Pix | [Mercado Pago](https://www.mercadopago.com.br) (API de Orders) |
| Lógica de backend (sem servidor próprio) | Supabase Edge Functions (Deno) |

### Por que essa arquitetura

O site é um único arquivo HTML autocontido (`index.html`), com CSS e JavaScript embutidos — sem build step, sem dependências de instalação. Toda a parte sensível (chaves secretas, processamento de pagamento, exclusão de conta) roda em Edge Functions no Supabase, nunca no código do navegador.

## 📂 Estrutura do repositório

```
index.html                  → aplicação principal (todo o site)
termos.html                 → Termos de Uso
politica-privacidade.html   → Política de Privacidade
```

## 🗄️ Banco de dados (Supabase)

Tabelas (schema `public`), todas com Row Level Security (RLS) ativado:

- **profiles** — dados do perfil de cada pessoa (nome, telefone, endereço, foto, status `activated`)
- **categories** — despesas cadastradas por cada pessoa (padrão + adicionadas manualmente)
- **month_entries** — valor de cada despesa em cada mês

Cada tabela tem políticas RLS garantindo que cada usuário só acesse (leia, edite, exclua) os próprios dados, usando `auth.uid()`.

## ⚡ Edge Functions (Supabase)

| Função | O que faz |
|---|---|
| `create-pix` | Gera uma cobrança Pix (via API de Orders do Mercado Pago) para o usuário autenticado |
| `mp-webhook` | Recebe a notificação do Mercado Pago quando o pagamento é aprovado e ativa a conta automaticamente |
| `delete-account` | Exclui completamente a conta do usuário (dados + login), usando a chave `service_role` |

Variáveis de ambiente (Secrets) necessárias nas Edge Functions:
- `MP_ACCESS_TOKEN` — token de produção do Mercado Pago
- `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY` — já fornecidas automaticamente pelo ambiente do Supabase

## 🔐 Segurança

- Senhas nunca passam pelo código do site — ficam só no Supabase Auth, com hash criptográfico
- Chaves sensíveis (Mercado Pago, banco de dados) existem apenas nas Edge Functions, nunca no front-end
- RLS garante isolamento de dados entre contas
- Entradas de texto do usuário (nome de despesas, perfil) passam por escape de HTML antes de serem exibidas, prevenindo XSS
- Site servido via HTTPS

## 🚀 Deploy

1. O conteúdo de `index.html`, `termos.html` e `politica-privacidade.html` é publicado diretamente via GitHub Pages (branch `main`, raiz do repositório).
2. O domínio customizado (`www.contasfixas.com.br`) está configurado via CNAME no DNS (registro.br) apontando para o GitHub Pages.
3. Qualquer atualização enviada a este repositório reflete automaticamente no site publicado — não há processo de build.

## 📧 Contato

sstecnologia.net@gmail.com
