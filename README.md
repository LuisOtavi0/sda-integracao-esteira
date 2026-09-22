================================================================================
TRABALHO PRÁTICO FINAL - SISTEMAS DISTRIBUÍDOS PARA AUTOMAÇÃO
SISTEMA DE SUPERVISÃO E TELEMETRIA DE ESTEIRA
================================================================================

Alunos:

- Luís Otávio de Souza Silva (2023082247)
- Maria Eduarda Neves Leal (2023111646)
  Disciplina: ELT011 - Sistemas Distribuídos para Automação (TN)
  Professor: Armando Alves Neto
  Universidade Federal de Minas Gerais (UFMG)

---

1. VISÃO GERAL DO SISTEMA

---

Este projeto simula e monitora uma esteira transportadora industrial em tempo real.
A arquitetura integra:

- Servidor OPC UA (Prosys) para disponibilização das variáveis de campo.
- Node-RED para aquisição, padronização, inferência de alarmes e Dashboard.
- Broker MQTT (Eclipse Mosquitto via Docker/WSL) para publicação da telemetria em JSON.

---

2. PRÉ-REQUISITOS E SOFTWARES NECESSÁRIOS

---

Para executar o ambiente, certifique-se de ter instalado no sistema:

1. Windows 10/11 com WSL2 (Ubuntu) e Docker Desktop configurado.
2. Prosys OPC UA Simulation Server (Versão Free ou Superior).
3. Node-RED rodando localmente (Node.js v18+ recomendado).
4. Nós do Node-RED instalados no seu ambiente:
   - node-red-contrib-opcua
   - node-red-dashboard

---

3. INSTRUÇÕES DE INSTALAÇÃO E CONFIGURAÇÃO

---

## PASSO 1: Subir o Broker MQTT (Mosquitto) no Docker (WSL)

No terminal do WSL Ubuntu ou PowerShell, execute o comando abaixo para iniciar
o contêiner do Mosquitto mapeado na porta 1883:

docker run -d --name mqtt-broker -p 1883:1883 -p 9001:9001 eclipse-mosquitto:latest

## PASSO 2: Configurar o Prosys OPC UA Simulation Server

1. Abra o Prosys OPC UA Simulation Server.
2. Certifique-se de que o servidor está rodando (Value Simulation = RUNNING).
3. Na aba 'Objects', crie a seguinte estrutura de pastas e variáveis:
   - Pasta: Processo/Motor
     - Motor_Estado (Boolean) -> NodeId: ns=3;i=1011
     - Motor_Velocidade (Float) -> NodeId: ns=3;i=1012
   - Pasta: Processo/Sensoriamento
     - Motor_Temperatura (Float) -> NodeId: ns=3;i=1013
     - Carga_Peso (Float) -> NodeId: ns=3;i=1014
4. Observe o Endpoint configurado (exemplo: opc.tcp://localhost:53530/OPCUA/SimulationServer).

## PASSO 3: Configurar e Importar o Fluxo no Node-RED

1. Inicie o Node-RED no terminal executando: node-red
2. Acesse o ambiente no navegador: http://localhost:1880
3. Instale as dependências (caso não tenha):
   - Menu Hambúrguer (Canto superior direito) -> Manage palette -> Install
   - Pesquise e instale: "node-red-contrib-opcua" e "node-red-dashboard".
4. Importe o fluxo do projeto:
   - Menu Hambúrguer -> Import -> Cole o código do arquivo 'flow.json' (disponível neste repositório) -> Import.
5. Verifique a URL do Endpoint OPC UA nos nós 'OpcUa-Client' para garantir que coincide com a porta exibida no seu Prosys.
6. Clique no botão "Deploy".

---

4. EXECUÇÃO E OPERAÇÃO DO SISTEMA

---

1. Visualização do Dashboard de Supervisão:
   - Abra o navegador no endereço: http://localhost:1880/ui
   - O painel exibirá em tempo real os ponteiros de carga e velocidade, gráficos de tendência temporal e sinalizadores de status do motor e alarmes.

2. Verificação do Envio de Mensagens MQTT:
   - No terminal do WSL, execute o comando subscriber para monitorar os tópicos:
     docker exec -it mqtt-broker mosquitto_sub -t "fabrica/#" -v
   - Verifique o recebimento contínuo das mensagens JSON estruturadas a cada 2 segundos.

3. Simulação de Alarmes para Testes:
   - No Prosys, altere a variável 'Motor_Temperatura' para um valor superior a 60.0 (°C) ou 'Carga_Peso' para um valor superior a 450.0 (kg) com o motor ligado.
   - Observe o Dashboard mudar imediatamente para "FALHA / ALARME ATIVO" e o payload MQTT registrar a chave de alarme ativada.

---

5. SUPORTE E CONTATO

---

Em caso de dúvidas na reprodução do ambiente, entre em contato com os autores
através dos e-mails institucionais da UFMG.
================================================================================
