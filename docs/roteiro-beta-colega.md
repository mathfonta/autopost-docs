# 📱 Roteiro completo — Beta test com o colega

> **Objetivo deste documento:** conteúdo pronto para você (Matheus) copiar e enviar ao seu colega, mais os cronogramas de acompanhamento. Complementa `docs/beta-onboarding-checklist.md` (que tem o checklist técnico interno) — este aqui é o material voltado para o colega.

---

## 1) Mensagem pronta para enviar (WhatsApp)

Copie, ajuste o tom se quiser, e envie em uma ou duas mensagens:

> Oi! Lembra que te falei do app que eu tô desenvolvendo? É um sistema que usa IA pra criar posts de Instagram pra empresas — você manda uma foto (de uma obra, imóvel, produto, sei do que for), a IA escreve a legenda, monta o design e sugere quando/como postar. Eu que fiz, e queria muito o seu olhar de fora antes de lançar de verdade.
>
> É um app que funciona direto no navegador do celular (não precisa baixar na loja) — te mando o passo a passo de como fixar ele na tela inicial, fica igual um app normal.
>
> Só uma coisa antes de você entrar: pra conseguir *conectar seu Instagram* e publicar de verdade (não só simular), eu preciso te adicionar como "testador" dentro das ferramentas de desenvolvedor da Meta — é uma exigência da própria Meta enquanto o app não passa pela revisão oficial deles, não tem nada a ver com dar acesso a dados sensíveis seus. Te explico certinho como funciona (é rapidinho).
>
> Se puder, me manda:
> 1. Seu **@ do Instagram** (o que você usa/usaria pra teste)
> 2. Confirma se essa conta é **Business/Creator** ou **pessoal** (se for pessoal, a gente troca — é grátis e leva 1 minuto)
>
> Assim que eu te adicionar como testador lá, te mando o link e o passo a passo. Bora? 🙌

---

## 2) O que é o AutoPost (explicação para o colega)

**Em uma frase:** um app que transforma uma foto em um post pronto para o Instagram — legenda, design e estratégia gerados por IA — com você só aprovando (ou ajustando) antes de publicar.

**Como funciona, na prática:**
1. Você tira ou escolhe uma foto (do seu negócio, produto, obra, imóvel — o que for).
2. Escolhe o formato: foto no feed, carrossel, Reels ou Stories.
3. A IA analisa a foto, escreve a legenda, monta o design (se for foto) e sugere uma estratégia de conteúdo.
4. Você revisa, edita se quiser, e aprova.
5. O post vai direto pro seu Instagram — publica agora ou agenda para um horário melhor.

**Não é preciso instalar nada de loja de aplicativo** — é um "PWA" (web app instalável): abre no navegador do celular e você fixa na tela inicial como um app comum (passo a passo abaixo).

---

## 3) Link de acesso

**https://autopost.app.br**

Funciona em qualquer navegador do celular. Recomendo:
- **iPhone:** abrir no **Safari** (obrigatório para o passo de instalação funcionar)
- **Android:** abrir no **Chrome**

---

## 4) Como fixar na tela inicial (passo a passo)

### 📱 iPhone (Safari)
1. Abra **https://autopost.app.br** no **Safari** (tem que ser o Safari, não funciona pelo Chrome/Instagram no iPhone)
2. Toque no ícone de **compartilhar** (o quadrado com uma flecha subindo), na barra de baixo
3. Role a lista de opções e toque em **"Adicionar à Tela de Início"**
4. Confirme o nome (**AutoPost**) e toque em **"Adicionar"**, no canto superior direito
5. Pronto — o ícone do AutoPost aparece na sua tela inicial, igual qualquer outro app

### 🤖 Android (Chrome)
1. Abra **https://autopost.app.br** no **Chrome**
2. O próprio Chrome pode mostrar um aviso automático perguntando **"Instalar app?"** — se aparecer, é só confirmar
3. Se não aparecer automaticamente: toque nos **três pontinhos** (menu, canto superior direito) → **"Adicionar à tela inicial"** ou **"Instalar app"**
4. Confirme
5. Pronto — o ícone do AutoPost aparece na sua tela inicial, igual qualquer outro app

Depois de instalado, ele abre em tela cheia, sem a barra de endereço do navegador — a experiência é a mesma de um app baixado da loja.

---

## 5) Por que precisamos do seu Instagram como "testador" na Meta

Isso costuma gerar dúvida, então vale explicar direito:

O AutoPost publica direto no Instagram através da API oficial da Meta (a empresa do Instagram/Facebook). Para qualquer app usar essa API de forma **liberada para o público em geral**, a Meta exige uma revisão formal chamada **App Review** — que ainda está em andamento para o AutoPost.

**Enquanto essa revisão não termina**, a Meta só permite que o app publique em contas que estejam explicitamente cadastradas como **"testador"** dentro do painel de desenvolvedor. É uma trava de segurança da própria plataforma, não uma escolha do AutoPost — e não dá acesso a nada da sua conta além do que você mesmo autorizar no momento de conectar (like qualquer login "Continuar com Instagram" que você já viu em outros apps).

**O que eu preciso de você:**
1. O seu **@ do Instagram** (username) — pra eu te adicionar como testador no painel da Meta
2. Você **aceitar o convite de testador** que vai chegar (Instagram → Configurações → Apps e sites → Convites de testador)
3. Confirmar que a conta é **Business ou Creator** — conta pessoal não consegue publicar via API. Se a sua for pessoal, é de graça e rápido converter: Instagram → Configurações → Conta → Mudar para conta profissional

Depois que você aceitar o convite, a conexão dentro do app é só um "Continuar com Instagram" normal — você autoriza, e pronto.

---

## 6) Sobre o feedback

O motivo principal de eu te chamar pra testar é ouvir sua visão de fora — de alguém que entende do assunto e não construiu o app comigo. Não existe resposta errada.

Ao longo da semana, vou te perguntar coisas como:
- A legenda que a IA escreveu ficou boa o suficiente pra você postar de verdade? O que você mudaria?
- O design serve pro seu negócio, ou precisaria de ajuste?
- Em algum momento você travou, ficou em dúvida, ou achou lento?
- Você pagaria por isso? Quanto pareceria justo por mês?
- Que 1 coisa, se existisse, te faria assinar na hora?

Pode falar comigo a qualquer momento durante a semana, não precisa guardar tudo pro final — se travar em algo, me chama na hora.

---

## 🗓️ Cronograma — Você (Admin)

| Quando | O que fazer |
|--------|-------------|
| **Antes de enviar** | Pedir @ do Instagram do colega e confirmar tipo de conta (Business/Creator) |
| **Antes de enviar** | Adicionar o colega como Instagram Tester no painel da Meta (`developers.facebook.com` → app AutoPost → App Roles → Roles) |
| **Antes de enviar** | Confirmar que `autopost.app.br` está no ar (checar `/health` ou abrir a página) |
| **Dia 1 (envio)** | Enviar a mensagem + este roteiro (seções 2 a 6) |
| **Dia 1-2** | Ficar disponível para dúvidas de instalação/onboarding/conexão do Instagram — é o ponto de maior risco de travar |
| **Dia 3** | Perguntar se ele já publicou algo; se não, entender o que travou |
| **Dia 4-5** | Deixar ele usar sozinho no dia a dia — evitar interromper demais, deixar a experiência natural aparecer |
| **Dia 6** | Enviar as perguntas de feedback (seção 6, ou a lista completa em `docs/beta-onboarding-checklist.md` Parte 5) |
| **Dia 7** | Consolidar o feedback recebido; registrar no Obsidian e/ou `docs/SESSAO-HANDOFF.md`; decidir o que entra na fila de produção |

## 🗓️ Cronograma — Colega (usuário beta)

| Quando | O que fazer |
|--------|-------------|
| **Dia 1** | Aceitar o convite de testador no Instagram; instalar o AutoPost na tela inicial (seção 4); criar conta; completar o onboarding (perfil da marca + conectar Instagram) |
| **Dia 2-3** | Publicar pelo menos 1-2 posts reais (foto de verdade do negócio dele) |
| **Dia 4-5** | Usar como usaria no dia a dia normal — se curioso, testar o agendamento (escolher publicar mais tarde em vez de agora) |
| **Dia 6** | Responder as perguntas de feedback que o Matheus enviar |

---

*Documento criado em 2026-07-26, complementar a `docs/beta-onboarding-checklist.md`.*
