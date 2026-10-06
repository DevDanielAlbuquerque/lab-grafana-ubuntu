# Laboratório de Linux e Grafana

Projeto de estudos em Linux e observabilidade, iniciado em uma VM local no VirtualBox e expandido para uma VM Ubuntu na Azure. O laboratório reúne Grafana, Prometheus e Node Exporter para acompanhar as métricas da própria máquina.

Trabalho com monitoramento no NOC e estou construindo minha base em Linux, redes e observabilidade. A proposta é entender cada etapa, praticar os comandos e documentar os resultados.

## Estado atual

**VM na Azure com métricas coletadas pelo Prometheus e visualizadas no Grafana.**

- [x] Instalar e acessar o Grafana na VM local.
- [x] Criar uma VM Ubuntu na Azure e acessar por SSH com chave `.pem`.
- [x] Atualizar o sistema e instalar o Grafana pelo APT.
- [x] Habilitar Grafana e Node Exporter no boot.
- [x] Instalar Node Exporter e Prometheus.
- [x] Validar a coleta com `up{job="node"} = 1`.
- [x] Conectar o Grafana ao Prometheus.
- [x] Criar painéis de CPU, memória, disco e tempo ligado.
- [x] Consultar os logs do Grafana com `journalctl`.

Também praticamos a configuração de um gráfico de histórico de CPU e memória. Loki, Alloy, banco de dados, alertas e teste de carga ficaram para possíveis etapas futuras.

O passo a passo com comandos e explicações está no [manual do laboratório](docs/manual-grafana-azure.md).

## Ambiente

| Componente | Primeira fase: local | Fase atual: Azure |
| --- | --- | --- |
| Infraestrutura | VirtualBox | VM na Azure, grupo de recursos `linux-estudos` |
| Sistema operacional | Ubuntu 22.04.5 LTS | Ubuntu 24.04 LTS (Noble) |
| Acesso administrativo | SSH pelo Windows | SSH pelo Windows com chave privada `.pem` |
| Visualização | Grafana OSS | Grafana OSS |
| Coleta | Ainda não implementada nessa fase | Prometheus e Node Exporter |
| Acesso web | IP da VM local, porta 3000 | Túnel SSH e `http://localhost:3000` |
| Instalação | APT | APT |

Os serviços foram instalados diretamente no Ubuntu. Não utilizamos containers nesta etapa. Região e tamanho da VM não estão registrados aqui; sua disponibilidade depende da assinatura e deve ser conferida no portal.

## Primeira fase: Grafana na VM local

1. Identifiquei o IP da VM com `ip address` e conectei pelo Windows usando SSH.
2. Consultei o disco com `df -h` e a distribuição com `lsb_release -a`.
3. Preparei as ferramentas de instalação e o diretório `/etc/apt/keyrings`.
4. Baixei a chave pública do Grafana para `/etc/apt/keyrings/grafana.asc` e configurei a permissão `644`.
5. Cadastrei o repositório em `/etc/apt/sources.list.d/grafana.list`.
6. Atualizei o catálogo do APT e instalei o pacote `grafana`.
7. Iniciei o serviço `grafana-server` e acessei a página pelo navegador do Windows.

Referência de instalação: [documentação oficial para Debian e Ubuntu](https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/).

## Evolução para Azure

1. Criei a VM e conectei pelo PowerShell usando a chave SSH.
2. Atualizei o Ubuntu e configurei a chave e o repositório oficial do Grafana.
3. Instalei o Grafana e habilitei o serviço `grafana-server`.
4. Testei a resposta local com `curl -I http://localhost:3000`, que retornou um redirecionamento HTTP 302.
5. Abri um túnel SSH no Windows e acessei a interface pelo navegador.
6. Instalei o Node Exporter e testei `http://localhost:9100/metrics`.
7. Instalei o Prometheus e conferi `/etc/prometheus/prometheus.yml`; o pacote já trouxe o job `node` configurado.
8. Validei pela API que os jobs `prometheus` e `node` estavam com `up = 1`.
9. Cadastrei `http://localhost:9090` como fonte Prometheus no Grafana; o teste retornou **Successfully queried the Prometheus API.**
10. Montei os indicadores da VM e acompanhei os registros de consultas do Grafana pelo terminal.

## Acesso ao laboratório na Azure

No **PowerShell do Windows**, substitua o caminho da chave e o IP:

```powershell
ssh -i "C:\caminho\chave-ubuntu-estudos.pem" -L 3000:localhost:3000 daniel@IP_PUBLICO_DA_VM
```

Mantenha a sessão aberta e acesse [http://localhost:3000](http://localhost:3000) no navegador. O túnel encaminha a porta 3000 do Windows para a porta 3000 da VM pela conexão SSH. Não é necessário abrir a porta 3000 publicamente para esse acesso.

A chave está no Windows: esse comando não deve ser executado dentro da VM usando um caminho `C:\...`. A chave `.pem`, a senha do Linux e a senha do Grafana têm funções diferentes. Use o IP atual, sem o sufixo de rede `/24`.

## Coleta e visualização

| Componente | Função | Porta |
| --- | --- | --- |
| Node Exporter | Expõe métricas do Linux em `/metrics` | 9100 |
| Prometheus | Busca as métricas e armazena o histórico | 9090 |
| Grafana | Consulta o Prometheus e apresenta os painéis | 3000 |

Os três serviços estão na mesma VM. O Prometheus consulta o Node Exporter; o Grafana consulta o Prometheus.

Configuração dos jobs presentes no laboratório:

```yaml
scrape_configs:
  - job_name: prometheus
    scrape_interval: 5s
    scrape_timeout: 5s
    static_configs:
      - targets: ['localhost:9090']

  - job_name: node
    static_configs:
      - targets: ['localhost:9100']
```

O job `node` herda o intervalo global de 15 segundos. Esse trecho fica dentro do arquivo completo, não substitui toda a configuração. Não foi necessário editar nem duplicar o job.

Para verificar a coleta, execute na VM:

```bash
curl -sG --data-urlencode 'query=up{job="node"}' http://localhost:9090/api/v1/query
```

O valor `1` confirma sucesso na última coleta do alvo; `0` indica falha na coleta.

## Painéis e consultas PromQL

Todos usam a fonte Prometheus e o filtro `job="node"`.

### Memória em uso (%)

```promql
100 * (1 - node_memory_MemAvailable_bytes{job="node"} / node_memory_MemTotal_bytes{job="node"})
```

Estima o percentual em uso a partir da memória disponível que o Linux informa. Visualização **Gauge**, unidade **Percent (0–100)**, limites 0 e 100.

### CPU em uso (%)

```promql
100 * (1 - avg by (instance) (
  rate(node_cpu_seconds_total{job="node", mode="idle"}[5m])
))
```

Calcula o percentual não ocioso médio dos núcleos, com janela de cinco minutos; inclui modos como espera por I/O. Visualização **Gauge**, unidade **Percent (0–100)**, limites 0 e 100.

### Disco em uso — raiz (%)

```promql
100 * (1 -
  node_filesystem_avail_bytes{job="node", mountpoint="/"}
  /
  node_filesystem_size_bytes{job="node", mountpoint="/"}
)
```

Mostra o percentual não disponível a usuários comuns no sistema de arquivos raiz, incluindo espaço reservado. Visualização **Gauge**, unidade **Percent (0–100)**, limites 0 e 100.

### Tempo ligado da VM

```promql
node_time_seconds{job="node"} - node_boot_time_seconds{job="node"}
```

Tempo desde o último boot, em segundos. Visualização **Stat**, unidade **Duration (s)**.

Nos indicadores, usar **Last (not null)** para apresentar o valor mais recente. Para comparar CPU e memória ao longo do tempo, usar as duas consultas em um painel **Time series**, tipo **Range**, com legendas `CPU` e `Memória`. O período sugerido é **Last 15 minutes**, com atualização de **15s**.

## Comandos de consulta e administração

Comandos de referência para o laboratório; a presença nesta lista não significa que todos os testes foram concluídos.

| Comando | Finalidade |
| --- | --- |
| `ip address` | Consultar os endereços das interfaces de rede |
| `df -h` | Consultar utilização e espaço disponível nos sistemas de arquivos |
| `lsb_release -a` | Consultar informações da distribuição |
| `pwd` | Mostrar o diretório atual |
| `ls` | Listar o conteúdo de um diretório |
| `sudo apt update` | Atualizar o catálogo de pacotes |
| `sudo apt install grafana` | Instalar o Grafana após configurar o repositório |
| `sudo systemctl start grafana-server` | Iniciar o serviço agora |
| `systemctl status grafana-server` | Consultar o estado do serviço |
| `sudo systemctl enable grafana-server` | Habilitar a inicialização no boot |
| `systemctl is-enabled grafana-server` | Conferir se a inicialização no boot está habilitada |

## Dificuldades e aprendizados

- **Pacote não encontrado:** tentar instalar o Grafana antes de configurar seu repositório retornou “Impossível encontrar o pacote grafana”. Aprendi a cadastrar a fonte e atualizar o catálogo antes da instalação.
- **Caminho relativo e absoluto:** `etc/apt/keyrings` dentro da pasta pessoal não é o mesmo que `/etc/apt/keyrings`. A barra inicial indica um caminho a partir da raiz.
- **Arquivo de chave:** `docker.asc` e `grafana.asc` são arquivos distintos. É necessário conferir o nome e o caminho antes de usar uma chave.
- **Sintaxe do repositório:** corrigi a escrita de `signed-by` e o espaço depois de `deb` na configuração do APT.
- **Baixar e instalar:** `wget` baixa arquivos; `apt install` instala pacotes e resolve suas dependências.
- **Redirecionamento e privilégios:** usei `echo` com `sudo tee` para gravar a configuração em um diretório do sistema.
- **Serviço e boot:** iniciar um serviço com `start` e habilitá-lo com `enable` são ações diferentes.

## Logs e próximos estudos

Consultamos os registros do serviço na VM:

```bash
sudo journalctl -u grafana-server -n 30 --no-pager
sudo journalctl -u grafana-server -f
```

O primeiro mostra as últimas trinta linhas; o segundo acompanha novos registros. Ctrl+C encerra o acompanhamento sem parar o Grafana.

Vimos consultas do plugin Prometheus com `endpoint=queryData`, `status=ok` e sua duração. São logs de operações do Grafana, não consultas SQL. Os logs continuam sendo consultados pelo terminal; não configuramos sua coleta no dashboard.

Possíveis próximos exercícios, ainda não realizados:

- Teste curto de carga e observação do efeito nas métricas.
- Coleta e visualização de logs com Alloy e Loki.
- Monitoramento de um banco de dados de laboratório.
- Configuração de alertas.

## Aprendizados da etapa Azure

- Executar comandos no computador certo: a chave privada estava no Windows, não na VM.
- Usar `sudo` nas operações administrativas do serviço.
- Distinguir pacote, serviço, chave do repositório e chave SSH.
- Validar cada camada: serviço ativo, endpoint respondendo, coleta com `up = 1` e conexão da fonte no Grafana.
- Entender que `localhost` depende de quem faz a conexão: Windows no navegador, VM na fonte de dados do Grafana.
- Diferenciar métricas numéricas de logs de eventos.
- Diferenciar frequência de coleta, atualização do dashboard e janela de cálculo da consulta.

## Método de estudo

Uso cursos da Alura e um laboratório prático para consolidar os fundamentos. A sequência de referência é Comunicação Web, Unix e Shell e Sistema Operacional Linux, com exercícios relacionados ao projeto.

Durante a prática, tento explicar ou montar o comando antes de consultar a resposta. Comandos novos são apresentados com sua finalidade e opções, e cada etapa é conferida antes de avançar.

## Evidências

O funcionamento dos serviços, da coleta e dos quatro indicadores foi validado durante a prática. As capturas do dashboard e dos testes ainda não foram adicionadas ao repositório. Antes de publicar imagens, revisar para não incluir senhas, tokens ou dados do trabalho.

## Autor

Daniel Alves de Albuquerque de Souza — estudos em Linux e observabilidade, com foco em evolução para SRE.
