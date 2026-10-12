# Automação n8n: Monitoramento de Logs de Equipamentos de Rede com Inteligência Artificial e Telegram

## 1. Visão Geral do Projeto
Esta automação tem como objetivo coletar, analisar e filtrar logs de equipamentos de rede (como roteadores, switches, firewalls) utilizando um Agente de IA para identificar anomalias, erros ou falhas críticas, disparando alertas automáticos via Telegram para a equipe de tecnologia.

---

## 2. Nós do n8n Utilizados e Suas Funções

Para estruturar este *workflow* de forma eficiente, utilizaremos os seguintes nós:

1. **Schedule Trigger (Gatilho de Agendamento)**
   * **Função:** Disparar o fluxo em horários específicos definidos pelo administrador (ex: a cada 1 hora, ou em horários críticos).

2. **SSH / Execute Command (ou Webhook / Syslog Receiver)**
   * **Função:** Acessar os equipamentos de rede via SSH (ou receber logs via Syslog/API) para extrair os registros de log recentes do sistema.

3. **Advanced AI / Basic LLM Chain (com Modelo de IA, ex: OpenAI, Anthropic, etc.)**
   * **Função:** Atuar como o "Agente de IA". O modelo recebe o bloco bruto de logs e aplica raciocínio para:
     * Filtrar ruídos e eventos normais.
     * Identificar logs de erro, alerta (`warning`), atenção, falha ou consumo excessivo de recursos (processamento/memória).
     * Estruturar a resposta em um formato claro e resumido para envio.

4. **If (Condicional)**
   * **Função:** Verificar a saída da IA. Se a IA indicar que há um problema real ou anomalia que exige atenção, o fluxo prossegue; caso contrário, o fluxo é encerrado sem enviar spam.

5. **Telegram (Node de Envio de Mensagem)**
   * **Função:** Enviar o relatório detalhado do erro/anomalia diretamente para o grupo do Telegram configurado para o Setor de Tecnologia.

---

## 3. Lógica de Funcionamento do Workflow (Passo a Passo)

1. **Disparador Temporal:** O **Schedule Trigger** ativa o fluxo nos horários programados.
2. **Coleta de Dados:** O nó de conexão (ex: **SSH**) acessa o equipamento de rede, executa o comando de leitura de logs (ex: `/log print` no MikroTik ou comandos equivalentes em outros vendors) e captura o texto bruto.
3. **Análise Inteligente (IA):** O bloco de texto coletado é enviado para o nó de IA junto com um *System Prompt* restritivo que define:
   * *O que ignorar:* Registros de rotina, conexões normais e eventos informativos.
   * *O que destacar:* Erros críticos, falhas de hardware, picos de uso de CPU/Memória e alertas de segurança.
4. **Filtro de Decisão:** O nó **If** analisa o campo de verificação retornado pela IA. 
   * Se `status_alerta = true`, o fluxo continua.
   * Se `status_alerta = false`, o fluxo para.
5. **Notificação:** O nó do **Telegram** formata a mensagem gerada pela IA e a entrega no grupo da equipe de tecnologia com o contexto e o detalhamento do problema.
