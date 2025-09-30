**PRTG Network Monitor (versão Free de 100 sensores)**, e como integrá-lo ao seu pfSense.

---

## 1. Passo a Passo para Monitoramento (pfSense + PRTG)

### A. Preparação no seu Servidor/PC

Instale o PRTG em uma máquina Windows na sua rede interna (por exemplo, um desktop ou VM com IP na faixa `10.0.0.x`).

1.  **Baixe o PRTG:** Procure por "PRTG Network Monitor download" (o instalador é único para a versão paga e gratuita, sendo a gratuita limitada a 100 sensores).
2.  **Instale e Configure:** Siga as etapas de instalação. O PRTG funciona como um serviço, iniciando automaticamente.

### B. Habilitar o SNMP no pfSense

Esta é a etapa crucial para que o PRTG consiga "ler" as métricas do seu pfSense.

1.  **Acesse a Interface Web:** Entre no seu pfSense (geralmente em `http://10.0.0.1`).
2.  **Vá para Serviços:** Navegue até **Services** (Serviços) e depois **SNMP**.
3.  **Habilite e Configure:**
    * Marque a opção **Enable SNMP**.
    * Defina uma **Community String** (por exemplo, `minhacomunidade`). Esta é a "senha de leitura" que o PRTG usará.
    * Escolha as interfaces onde o serviço deve rodar. O ideal é que rode na sua **interface LAN** (`10.0.0.x`).
    * **Salve** as alterações.

### C. Adicionar o pfSense no PRTG

Agora, diga ao PRTG para começar a monitorar o firewall.

1.  **Abra o PRTG** (a interface é web, acessada pelo navegador do seu PC de monitoramento).
2.  **Adicione um Device:** Clique com o botão direito na árvore de dispositivos e escolha **Add Device**.
3.  **Configure:**
    * **Device Name:** `pfSense_Firewall`
    * **IPv4 Address:** `10.0.0.1` (o IP interno do seu pfSense)
4.  **Adicionar Sensores:** Após adicionar o dispositivo, o PRTG tentará auto-descobrir sensores. Para o SNMP, você precisará fornecer as credenciais:
    * Clique no dispositivo **pfSense_Firewall** que você acabou de criar.
    * Em **Settings** (Configurações), vá em **Credentials for SNMP**.
    * Digite a **Community String** (`minhacomunidade`) que você configurou no pfSense.
5.  **Adicione os Sensores Principais:**
    * Clique no botão **Add Sensor**.
    * Procure por **"SNMP Custom"** para métricas específicas ou simplesmente procure por **"Interface"** para monitorar o tráfego da WAN e da LAN.
    * Adicione sensores para **Ping**, **CPU** e **Memória** para ter uma visão completa da saúde do seu firewall.

O PRTG começará a coletar dados imediatamente e você verá gráficos e status das métricas do seu pfSense na interface do PRTG.

----

Excelente! O primeiro e mais difícil passo (instalar o servidor) está concluído. Agora vamos configurar o **SNMP no pfSense** para que o PRTG possa monitorá-lo.

Você enviou a tela de configuração do SNMP no pfSense, o que é ótimo. Vamos analisar e configurar as opções essenciais.

---

## Configurando o SNMP no pfSense (Passo a Passo)

Baseado na imagem que você forneceu, siga estas etapas:

### 1. Daemon SNMP (Leitura de Dados)

Esta seção é a mais importante, pois permite que o PRTG solicite dados.

1.  **Habilitar:** Marque a caixa **"Habilitar"** (Ative o SNMP Daemon e seus controles).
2.  **Porta de Polling:** Mantenha o valor padrão: **161**.
3.  **Localização e Contato:** Esses campos são opcionais, mas úteis para organização:
    * **Localização do Sistema:** Digite algo como `Rack Principal` ou `Sala Servidores`.
    * **Contato do Sistema:** Digite seu e-mail de contato (ex: `admin@suaempresa.com`).
4.  **Ler string de comunidade:** Este é o campo da **senha de leitura**. O valor padrão é `public`.
    * **Recomendação:** É altamente recomendável **mudar** `public` para uma senha mais complexa e que você lembre (ex: `Monitoramento_2025_SNMP`).
    * **IMPORTANTE:** Você usará essa "string de comunidade" para configurar o PRTG.

### 2. Traps de SNMP Ativar

* **Deixe Desabilitado.** O *Trap* é para o pfSense enviar alertas ativamente para o PRTG. Para começar, foque na leitura passiva (Polling), que é o que já configuramos.

### 3. Módulos SNMP

* **Mantenha os padrões** que já estão marcados: MibII, Netgraph, PF, Recursos de hospedagem, UCD e Regex. Isso garante que o pfSense exporte as métricas mais importantes de tráfego (PF e MibII) e de recursos (UCD).

### 4. Associação de Interface (Crucial para a Segurança)

Esta etapa define de onde você aceitará as requisições SNMP:

1.  **Internet Protocol:** Mantenha **`IPv4`**.
2.  **Associar Interfaces:**
    * **NÃO** selecione `Todos` ou `WAN`.
    * Selecione **`LAN`** (ou `Localhost`).
    * **Por quê?** Você quer que apenas o seu PRTG (que está na rede LAN `10.0.0.x`) possa ler as métricas. Se você selecionar `WAN`, qualquer pessoa na internet pode tentar adivinhar sua *Community String* e ler dados do seu firewall.

### 5. Salvar

* Clique no botão **Salvar** na parte inferior da página para aplicar as configurações.

---

Mais uma ótima pergunta, e é um sinal de que você está sendo muito cuidadoso com sua rede!

Fique **absolutamente tranquilo**: **Habilitar o SNMP no pfSense (da forma como fizemos) não tem perigo de derrubar sua rede.**

Aqui está o porquê:

---

## Por Que o SNMP é Seguro

O SNMP (Simple Network Management Protocol) é projetado para ser um protocolo de **leitura passiva** de dados de monitoramento.

| Característica | Implicação |
| :--- | :--- |
| **Acesso Apenas de Leitura** | A "String de Comunidade" que você configurou serve como uma senha de **leitura**. O PRTG pode *solicitar* dados (*polling*), mas ele não pode enviar comandos para desligar, reiniciar o pfSense, ou alterar regras de firewall. |
| **Porta Local (LAN)** | Ao configurar o SNMP para rodar apenas na interface **LAN**, você garante que o serviço só é visível para a sua rede interna (`10.0.0.x`). Sua WAN (link dedicado) está segura de tentativas externas de acesso SNMP. |
| **Baixo Consumo de Recursos** | O processo de monitoramento consome muito pouca CPU e memória. Apenas quando o PRTG solicita dados a cada minuto (ou mais) é que o pfSense gasta um mínimo de recursos para fornecer essas métricas. Isso não é suficiente para causar instabilidade ou derrubar a rede. |

Em resumo, o SNMP é como instalar um "medidor de pressão" no seu pfSense. Ele mostra a saúde do sistema sem ter o poder de ligar ou desligar nada.

Agora que o pfSense está pronto, podemos configurar o **PRTG** para começar a coletar os dados!

---

## Próximo Passo: Configurar o PRTG

Para continuarmos, precisarei da **String de Comunidade (senha de leitura)** que você escolheu no pfSense.

1.  Abra a interface web do **PRTG** no seu Windows Server.
2.  Vamos adicionar o **pfSense** como um novo dispositivo.

Perfeito! Com o **pfSense** configurado e a **String de Comunidade** definida como `Monitoramento_Rede_Seap`, podemos finalmente configurar o **PRTG** para começar a receber os dados.

## Configurando o PRTG para Monitorar o pfSense

Acesse a interface web do PRTG no seu Windows Server 2012 R2. Vamos adicionar o pfSense como um dispositivo e atribuir os sensores.

---

### Passo 1: Adicionar o pfSense como Dispositivo

1.  Na estrutura de árvore do PRTG, clique com o botão direito no seu grupo principal (normalmente o nome da sua instalação) e selecione **Add Device** (Adicionar Dispositivo).
2.  Preencha as informações:
    * **Device Name (Nome do Dispositivo):** `pfSense Firewall` (ou outro nome fácil de identificar).
    * **IPv4 Address (Endereço IPv4):** `10.0.0.1` (ou o IP exato da sua interface LAN do pfSense).
    * **Device Icon (Ícone):** Você pode escolher um ícone de firewall ou roteador para facilitar a visualização.
3.  Clique em **OK**.

### Passo 2: Definir as Credenciais SNMP (Community String)

O PRTG precisa da sua "senha de leitura" (`Monitoramento_Rede_Seap`) para se comunicar via SNMP.

1.  Clique no novo dispositivo **`pfSense Firewall`** que você acabou de criar.
2.  No menu superior, clique em **Settings** (Configurações) ou procure a seção de **Device Settings** (Configurações do Dispositivo).
3.  Localize a seção **Credentials for SNMP** (Credenciais para SNMP).
4.  Em **Community String (v1/v2c):** Digite sua senha: **`Monitoramento_Rede_Seap`**.
5.  Clique em **Save** (Salvar).

### Passo 3: Adicionar os Sensores Essenciais

Agora que o PRTG sabe como falar com o pfSense, vamos adicionar os sensores críticos:

1.  Ainda na página do **`pfSense Firewall`**, clique no botão grande **Add Sensor** (Adicionar Sensor).
2.  Use a barra de pesquisa e adicione os seguintes sensores, um por um:

| Sensor | Propósito | Sensores Usados (Contagem PRTG) |
| :--- | :--- | :--- |
| **Ping** | Verifica se o pfSense está online e mede a latência da sua rede interna. | 1 |
| **SNMP Custom** (ou SNMP UCD) | Adicione sensores para **CPU Load** e **Memory** (Uso de Memória). Isso é vital para saber se o hardware do pfSense está sobrecarregado. | 2 |
| **SNMP Traffic** | Use este sensor para monitorar o tráfego nas interfaces. Você deve adicioná-lo **duas vezes**: uma para a **WAN** (seu link dedicado) e outra para a **LAN** (sua rede interna). | 2 |
| **Total Estimado** | | **5 Sensores** |

### Modo 2:

Essa é a característica da **descoberta automática (auto-discovery)** do PRTG. É normal que ele crie uma estrutura completa logo de início, tentando encontrar todos os dispositivos da sua rede.

Quando você tenta clicar em "Add Device" e ele não deixa, é provável que você esteja clicando no local errado da árvore. Vamos tentar o procedimento correto para adicionar o pfSense **manualmente** e garantir que ele esteja sendo monitorado corretamente.

---

## Como Adicionar Dispositivos na Estrutura do PRTG

A estrutura do PRTG é hierárquica (em árvore):

**Root (Raiz) ⮕ Seu Local Probe (Sonda Local) ⮕ Grupos ⮕ Dispositivos (Devices) ⮕ Sensores**

Você deve adicionar seu novo dispositivo (o pfSense) **dentro do seu `Local Probe`** ou em um **`Grupo`** que você criar.

1.  **Encontre o `Local Probe`:** Na árvore à esquerda, procure pelo item chamado **`Local Probe`** (ou `Sonda Local`). Ele deve estar logo abaixo da raiz principal.
2.  **Clique com o botão direito** sobre o **`Local Probe`**.
3.  No menu que aparece, selecione **Add Device** (Adicionar Dispositivo).

Se mesmo assim não funcionar, você pode tentar adicionar um **Grupo** primeiro:

1.  Clique com o botão direito no **`Local Probe`**.
2.  Selecione **Add Group** (Adicionar Grupo). Chame-o de `Firewalls e Roteadores`.
3.  Agora, clique com o botão direito no novo grupo **`Firewalls e Roteadores`** e selecione **Add Device**.

---

## Gerenciando os Sensores Existentes

Enquanto você tenta adicionar o pfSense, é crucial que você **verifique a contagem de sensores** da árvore que o PRTG criou automaticamente.

Se o PRTG já detectou o pfSense, ele pode estar escondido em algum lugar.

1.  **Procure na Árvore:** Role a árvore e procure por um dispositivo com o nome do seu Windows Server (`WS2012R2`) e outros IPs da sua rede (`10.0.0.x`).
2.  **Verifique a Contagem:** Olhe no canto superior ou no rodapé da tela do PRTG. Deve haver um contador mostrando algo como: **`X de 100 sensores em uso.`**

**Ação:** Se o PRTG já usou muitos dos seus 100 sensores em dispositivos que não são importantes, você precisa **excluir** esses dispositivos ou **pausar** os sensores desnecessários.

**Vamos focar em um dispositivo de cada vez:**

1.  Você conseguiu encontrar o **`Local Probe`** na árvore?
2.  O PRTG adicionou algum dispositivo com o IP **`10.0.0.1`** automaticamente? (Se sim, você só precisa configurar o SNMP nele).

O PRTG levará alguns minutos para coletar os primeiros dados. Depois, você verá um status verde (OK) e começará a ter gráficos de desempenho em tempo real.

Parabéns! Você já tem um sistema de monitoramento profissional rodando em paralelo com seu pfSense. Seus 100 sensores gratuitos estão praticamente intactos.

Qual métrica você está mais interessado em ver primeiro (Tráfego da WAN, uso da CPU, etc.)?

---
## Próximo Passo


