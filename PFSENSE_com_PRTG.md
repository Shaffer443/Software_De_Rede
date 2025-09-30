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

Se precisar de ajuda com a instalação de algum pacote específico ou com uma das etapas acima, me avise!
