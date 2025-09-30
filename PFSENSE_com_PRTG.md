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

## Próximo Passo

O SNMP está liberado e pronto no pfSense. Agora precisamos ir para o PRTG para criar o dispositivo e adicionar os sensores.

Qual foi a **String de Comunidade** que você escolheu? (Você precisará dela agora.)
