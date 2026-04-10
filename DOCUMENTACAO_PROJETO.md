# TeepSaude — Painel clínico e ecossistema (documento do projeto)

Este documento descreve **o que o produto pretende fazer**, **como as partes se relacionam** e **como a integração com o app mobile** se encaixa. O HTML em `index.html` nesta pasta é um **protótipo de demonstração** (dados em memória); o comportamento real dependerá de backend, autenticação e APIs.

### Natureza do produto: TeepSaude como **SaaS** (sistema futuro)

O ecossistema TeepSaude — **servidor**, **APIs**, **aplicativo do titular**, **painéis web** (clínica e, quando existir, área pessoal) — está especificado e será ofertado como **SaaS** (*Software as a Service*), ou seja:

- **Hospedagem e operação** pela empresa provedora (nuvem), não instalação on-premise por padrão;
- **Acesso** via internet (web e app), com autenticação e políticas de segurança definidas no contrato;
- **Atualizações de produto** centralizadas para todos os clientes, com roadmap comum;
- **Dados** do titular e **configurações** das organizações (ex.: clínicas) mantidos em infraestrutura do serviço, com **isolamento lógico** entre **tenants** (multi-tenant), conforme LGPD e termos de uso;
- **Modelo comercial** típico de SaaS: assinatura, licença por organização/usuário ou consumo — a definir comercialmente.

Este documento refere-se a esse **sistema futuro** enquanto produto **SaaS**. O protótipo em `index.html` **não** constitui o SaaS em si: apenas simula telas; a camada de serviço real será backend, filas, identidade e APIs descritas nas seções seguintes.

---

## 1. Cenário central: dados no SaaS e vínculo explícito

### 1.1 Paciente / titular só no app

- Existe **usuário que usa apenas o app TeepSaude** (e, no futuro, a **área web pessoal**).
- Os **dados de saúde** que ele registra (sinais vitais, hábitos, medicações, etc.) são **armazenados no servidor (SaaS)**, associados à **conta dele**.
- **Enquanto não houver vínculo explícito** com uma clínica (ou outro **grupo gerenciador**), essa clínica **não vê, não analisa e não sincroniza** esses dados — mesmo que existam no servidor.

### 1.2 O que é “vínculo”

- **Vínculo** é a relação autorizada **titular ↔ grupo** (ex.: clínica, núcleo familiar).
- O titular **aceita** no app (ou canal seguro equivalente) que determinado gerenciador acesse **um escopo** de dados (o que a política de produto definir).
- Sem esse **consentimento**, o painel da clínica não deve tratar esse usuário como “paciente visível” com os mesmos dados do app.

### 1.3 Dois modos no ecossistema web

| Módulo | Público | Papel |
|--------|---------|--------|
| **Área pessoal** (web) | Titular | Gerenciamento dos **próprios** dados; sem obrigatoriedade de clínica. |
| **Painel do gerenciador** (esta linha de produto) | Clínica / familiar autorizado | Carteira, alertas, agenda, análise — **após vínculo** e conforme permissões. |

O `index.html` atual simula sobretudo o **painel da clínica** (carteira, alertas, agenda, cadastro com convite ao app, etc.).

---

## 2. O que este projeto (pasta `clinica`) faz hoje (demo)

- **Carteira de pacientes** com KPIs, busca, tabela e coluna lateral com resumo de **alertas** e **agenda do dia**.
- **Perfil do paciente** com abas (Saúde, Corpo, Medicação, Histórico, Contato), histórico de medições em modal, chat simulado, agendamento.
- **Análise de alertas** (menu Análise) com filtros por tipo, paciente, período e texto.
- **Agenda** (hoje / amanhã) alinhada ao discurso de produto de sincronizar com o app do paciente.
- **Cadastro de paciente** na clínica com opção de **conta no app** (usuário + senha provisória, troca no 1º acesso) e **convite via WhatsApp** com link de download — **somente demonstração**; em produção: hash de senha, canais seguros, LGPD.
- Navegação por **sidebar** (Pacientes, Agenda; Análise: Alertas, Relatórios, Exames, Medicações).

Limitações explícitas da demo: sem API real, sem multi-tenant, sem fila de “vínculo pendente” ligada ao app.

---

## 3. O que o produto completo deve fazer (alvo)

### 3.1 Cadastrar vs vincular

- **Cadastrar titular / paciente na clínica**: cria ficha no **grupo** clínica; pode incluir criação de credenciais de app e convite (fluxo já esboçado na demo).
- **Vincular titular existente**: a clínica informa um **código de vínculo** exibido no app pelo titular; o sistema dispara **pedido de autorização**; o titular **aprova no app**; só então os dados do SaaS ficam acessíveis ao painel daquela clínica no escopo acordado.

### 3.2 Grupos e papéis

- Uma **organização / grupo** (ex.: Clínica TeepSaude) agrega **membros** (médicos, recepção) com papéis e permissões.
- Outros tipos de grupo (ex.: familiar monitorando titular) seguem a **mesma ideia de vínculo + escopo**, com regras diferentes.

### 3.3 Notificações

- App: notificação de **“Clínica X pede acesso aos seus dados”** com aceitar / recusar.
- Clínica: estados **pendente**, **ativo**, **recusado**, **revogado** na UI de vínculos.

---

## 4. Integração com mobile (app TeepSaude)

### 4.1 Fonte da verdade

- O **servidor SaaS** é a fonte de verdade dos dados do titular.
- O **app** grava e lê via API autenticada (token da conta).

### 4.2 Sincronização com a clínica

- **Com vínculo ativo**: o painel clínico consome APIs **autorizadas** (leitura/escrita conforme contrato de escopo) sobre os mesmos recursos que o app usa onde aplicável (ex.: agenda, parte dos sinais vitais, alertas gerados por regras clínicas).
- **Sem vínculo**: APIs da clínica **não** retornam dados clínicos desse titular; apenas fluxos de **convite** ou **pedido de vínculo** podem existir.

### 4.3 Autenticação e primeiro acesso

- Credenciais criadas na clínica (demo): usuário + **senha provisória**; no **primeiro login** o app exige **nova senha**.
- Em produção: fluxo OAuth2/OpenID ou JWT, política de senha, recuperação segura, auditoria.

### 4.4 Identificadores

- **ID de conta** (titular) no SaaS.
- **Código de vínculo** (temporário, de uso único ou com TTL) gerado no app para a clínica digitar no painel — detalhe de implementação a definir com segurança.

### 4.5 Canais de convite (produto)

- WhatsApp / SMS / e-mail como **atalhos** para o titular instalar o app ou abrir o link de aceite; o **aceite legal e técnico** continua sendo no app ou fluxo autenticado.

---

## 5. Arquitetura lógica (visão resumida)

```
[ App mobile TeepSaude ] ←→ [ API SaaS — dados do titular ]
                                    ↑
                                    │ vínculo + escopo
                                    ↓
[ Painel web clínica ]     ←→ [ API — visão do grupo + políticas ]
[ Área web pessoal ]       ←→ [ API — só titular ]
```

---

## 6. Glossário alinhado (referência rápida)

| Termo | Uso |
|--------|-----|
| **SaaS (Software as a Service)** | Modelo em que o TeepSaude é **fornecido como serviço na nuvem**: o cliente usa app e painéis via internet; infraestrutura, atualizações e persistência ficam com o provedor, no marco contratual e legal (LGPD). |
| **Titular** | Dono da conta e dos dados de saúde no SaaS. |
| **Conta** | Identidade no TeepSaude (login + dados associados). |
| **Grupo / organização** | Clínica, família, etc., que pode gerenciar titulares **vinculados**. |
| **Gerenciador** | Quem age em nome do grupo no painel web. |
| **Vínculo** | Autorização titular → grupo, com escopo e estado (pendente, ativo, …). |
| **Código de vínculo** | Código que o titular fornece à clínica para iniciar o pedido de acesso. |

---

## 7. Próximos passos sugeridos

1. **Alinhar o esboço da demonstração** (`index.html`) com os fluxos: tela “Vincular titular”, estados pendentes, e cópias de UI usando o glossário acima.
2. Definir **contrato de API** mínimo (auth, vínculo, leitura de paciente vinculado).
3. Especificar **LGPD**: bases legais, registro de consentimento, revogação e retenção.

---

*Documento gerado para alinhamento de produto. Ajuste datas, nomes de módulos e URLs de app conforme a versão comercial do TeepSaude.*
