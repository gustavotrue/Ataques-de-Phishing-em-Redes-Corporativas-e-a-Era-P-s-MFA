# Ataques-de-Phishing-em-Redes-Corporativas-e-a-Era-P-s-MFA
Este repositório contém a curadoria de fontes, a documentação de engenharia de prompts ("cicatrizes" de testes) e o miniguia consolidado de estudos sobre campanhas modernas de phishing corporativo, com foco em bypass de MFA.

**Contexto e Objetivos**
O phishing continua sendo o vetor de ataque inicial mais comum e eficaz para comprometer redes corporativas. Tradicionalmente, as organizações confiavam na Autenticação de Múltiplos Fatores (MFA) baseada em SMS ou aplicativos de autenticação (TOTP/Push) como uma barreira definitiva. No entanto, o cenário de ameaças evoluiu drasticamente.

Este caderno temático investiga as técnicas modernas de evasão de MFA, especificamente:

- Adversary-in-the-Middle (AiTM): Uso de proxies reversos para capturar cookies de sessão ativa em tempo real.

- Abuso de Device Code Flow: Exploração de fluxos legítimos de autenticação de dispositivos que não possuem navegadores convencionais.


**Objetivos de Estudo:**

1. Compreender o funcionamento técnico das campanhas de phishing modernas que contornam o MFA tradicional.

1. Analisar táticas reais de operadores de ameaças a partir de falhas operacionais (OPSEC) expostas no mercado.

3. Documentar estratégias robustas de detecção e mitigação no nível de rede, endpoint e gerenciamento de identidade (como Microsoft Entra ID).


**Curadoria de Fontes**

Para alimentar seu NotebookLM, recomendamos o upload dos seguintes documentos (as fontes sugeridas cobrem desde teoria até casos práticos):

1. Fonte Prática (Caso de Estudo Principal):
   - Título: Um Servidor Mal Configurado Expôs Três Operações de Phishing Contra o Microsoft 365.
   - Origem/Link: XPSec Security Blog.
   - Descrição: Análise forense de uma falha de OPSEC de atacantes que expôs um servidor Python com diretórios abertos e histórico de comandos em Budapeste. Detalha o uso de ferramentas como Evilginx, MaDoO Blaster e persistência usando XEOX RMM.

2. **Guia de Mitigação da Agência de Segurança Governamental (CISA)**
   - Fonte: Capacity-Enhancing Guide: Preventing and Mitigating Advanced Phishing Attacks (PDF Oficial)
   - Link de Acesso: CISA capacity-enhancing-guide-preventing-and-mitigating-advanced-phishing-attacks.pdf
   - Foco no NotebookLM: Esta diretriz oficial da agência de segurança dos EUA detalha exatamente o que constitui a autenticação resistente a phishing (FIDO2/Passkeys) e como organizações devem arquitetar suas defesas.
  
3. Mapeamento de Táticas, Técnicas e Procedimentos (TTPs)
   - Fonte: MITRE ATT&CK - Technique T1566: Phishing
   - Link de Acesso: MITRE ATT&CK Enterprise Matrix (T1566)
   - Foco no NotebookLM: Estruturar o conhecimento teórico no formato de matriz corporativa. Útil para quando você pedir ao NotebookLM para mapear um ataque contra o framework oficial utilizado pelo mercado de segurança.

4. Análise Técnica de Ferramentas Ofensivas (Evilginx/AiTM)
   - Fonte: GitHub Repository & Documentation - Evilginx 3
   - Link de Acesso: Evilginx 3 Official Repository by KGloss7 (ou a documentação oficial em evilginx.com)
   - Foco no NotebookLM: Permitir que a IA entenda a mecânica por trás do proxy reverso de forma extremamente detalhada — como os phishlets (arquivos de configuração de domínios específicos como o do Microsoft 365) são estruturados para capturar tokens.


**Engenharia de Prompts e "Cicatrizes" (Troubleshooting)**
Ao interagir com o NotebookLM, o maior desafio é evitar respostas genéricas e extrair detalhes operacionais granulares. Abaixo estão documentados os testes de prompts aplicados durante a fase de desenvolvimento deste estudo.

Prompt de Teste 1 (Abordagem Genérica - Falhou em trazer detalhes profundos)
Prompt: "Como funciona o phishing contra o Microsoft 365?"
  - Resultado: Resposta superficial. Explicou o conceito básico de e-mails falsos solicitando senhas, sem mencionar cookies de sessão ou fluxos OAuth.
  - Cicatriz/Dificuldade: A IA tendeu a focar no phishing tradicional ("antigo") e ignorou as técnicas modernas de bypass de MFA.

Prompt de Teste 2 (Refinado com Contexto e Engenharia de Prompt)
Prompt: "Com base no artigo da XPSec, explique detalhadamente a diferença técnica no fluxo de autenticação entre um ataque AiTM com Evilginx e o abuso do Device Code Flow. Indique como cada um interage com o usuário e onde os tokens são interceptados."
  - Resultado: Resposta técnica precisa. A IA mapeou o proxy reverso do Evilginx capturando o cookie de sessão e o abuso do fluxo genuíno [microsoft.com/devicelogin](https://microsoft.com/devicelogin) no Device Code Flow, onde o usuário insere um código gerado pelo atacante.
  - Aprendizado (Troubleshooting): Forçar a IA a contrastar duas técnicas e nomear ferramentas específicas do material de origem faz com que ela extraia as nuances técnicas reais do texto de apoio.


**Miniguia de Estudo (Entrega Final)**
Este é o compilado do conhecimento extraído do caso de estudo e das melhores práticas de segurança ofensiva e defensiva

Resumos Estruturados do Assunto
O Caso do Servidor de Phishing Exposto (XPSec Security)
Em abril de 2026, pesquisadores identificaram um servidor de phishing hospedado na Hungria com a listagem de diretórios ativa (directory listing) e o histórico de comandos do terminal visível. Isso revelou três operadores ativos (codemado, mail-argenta e saroula01) que coletavam credenciais de organizações nas Américas e na Europa. A análise dos logs mostrou um alto nível de persistência, com tokens de acesso configurados para se autorrenovarem por até um ano, mantendo o acesso silencioso mesmo após mudanças de senha padrão do usuário.

Técnica 1: Adversary-in-the-Middle (AiTM) com Evilginx
  - Mecanismo: O atacante atua como um intermediário (proxy) entre a vítima e o portal real da Microsoft.
  - O Bypass do MFA: Como a vítima está enviando os dados para a Microsoft por meio do proxy, o MFA é solicitado e resolvido legitimamente pela própria vítima. Após o sucesso, a Microsoft gera um cookie de sessão válido. O proxy intercepta este cookie.
  - Persistência: De posse do cookie de sessão, o atacante clona a sessão autenticada em seu próprio navegador sem passar pelo fluxo de autenticação novamente.

Técnica 2: Abuso de Device Code Flow
  - Mecanismo: O atacante inicia um fluxo de login projetado para dispositivos inteligentes (como TVs) e obtém um código exclusivo. Através de engenharia social, induz a vítima a acessar a URL real e segura da Microsoft ([microsoft.com/devicelogin](https://microsoft.com/devicelogin)) e inserir o código.
  - Por que funciona: Não há site falso. A vítima confia no domínio da Microsoft. Ao digitar o código e autorizar, ela involuntariamente concede acesso total aos recursos da sua conta para o dispositivo/terminal controlado pelo atacante.


**Táticas de Mitigação e Defesa**
A defesa deve ser desenhada para neutralizar os vetores específicos de cada técnica de ataque:

_Vetor de Ataque:_
Phishing AiTM (Evilginx)
_Medida Preventiva Principal:_
Autenticação Resistente a Phishing (FIDO2 / Passkeys): Chaves criptográficas atreladas ao domínio original. O navegador se recusa a assinar o desafio de login se o site for um proxy/domínio incorreto.
_Detecção Ativa:_
Buscar logins com renovações de token excessivas em geolocalizações anômalas nos logs de identidade.

_Vetor de Ataque:_
Device Code Flow
_Medida Preventiva Principal:_
Políticas de Acesso Condicional (Conditional Access): Restringir ou bloquear totalmente o uso do Device Code Flow para usuários comuns que não necessitam desse tipo de login.
_Detecção Ativa:_
Monitorar nos logs do Entra ID o Client ID d3590ed6-52b3-4102-aeff-aad2292ab01c (associado ao fluxo do Office) vindo de IPs desconhecidos.

_Vetor de Ataque:_
Persistência em Endpoint
_Medida Preventiva Principal:_
Bloqueio de ferramentas de RMM (Remote Monitoring and Management) não autorizadas.
_Detecção Ativa:_
Buscar no endpoint pelo binário do agente xeox-agent_x64.exe e tarefas agendadas sob o padrão *XEOX*Agent*Watchdog*.


**Glossário de Conceitos Aprendidos**
  - AiTM (Adversary-in-the-Middle): Um tipo de ataque onde o adversário se posiciona entre duas partes legítimas para interceptar e, se necessário, modificar a comunicação. No contexto web moderno, refere-se a proxies reversos que roubam credenciais e cookies de sessão em tempo real.
  - Cookie de Sessão: Arquivo de dados gerado por um site após a autenticação bem-sucedida, que serve como uma "chave de identificação temporária" para que o usuário não precise digitar sua senha a cada nova página acessada.
  - Device Code Flow (Fluxo de Código do Dispositivo): Um fluxo de concessão do protocolo OAuth 2.0 projetado para dispositivos com conexões de internet limitadas ou sem interfaces de entrada de texto confortáveis.
  - FIDO2 / Passkeys: Padrões de autenticação baseados em criptografia de chave pública que garantem resistência nativa a ataques de phishing, pois vinculam a credencial ao domínio exato do serviço legítimo.
  - Continuous Access Evaluation (CAE): Recurso que permite que serviços de nuvem (como Microsoft 365) reavaliem em tempo real as políticas de segurança e revoguem sessões ativas imediatamente se houver alterações de risco (ex: mudança de IP ou alteração de senha).
  - OPSEC (Operations Security): Processo analítico que ajuda a identificar informações críticas e proteger ações que possam ser úteis para adversários. No caso analisado, a má OPSEC do atacante permitiu que suas táticas fossem desvendadas.


**Conjunto de Prompts Reutilizáveis (Para revisões futuras)**
Guarde estes prompts para interagir com o seu NotebookLM à medida que adicionar novos arquivos PDF e relatórios de inteligência sobre ameaças:

1. Análise de novos incidentes baseada em histórico:
   - "Com base nos novos relatórios anexados, há indícios de que os atacantes estão usando técnicas de desvio de MFA parecidas com as campanhas de AiTM (Evilginx) ou Device Code Flow documentadas pela XPSec em 2026? Identifique pontos em comum nas ferramentas e infraestrutura."
  
2. Revisão de regras de detecção de logs:
   - "Estou criando uma regra de correlação SIEM para detectar abuso de Device Code Flow. Com base na nossa base de conhecimento, quais são os Client IDs específicos da Microsoft, strings em campos de métodos de transferência ou eventos de endpoint que eu preciso monitorar?"
  
3. Simulação de Red Team:
   - "Aja como um especialista em Segurança Ofensiva. Desenhe um plano de teste de invasão controlado (Red Team) para avaliar a resiliência de nossa organização contra ataques Adversary-in-the-Middle, especificando como testar se nossas chaves FIDO2 estão configuradas corretamente."
