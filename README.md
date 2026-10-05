# Laboratório de Linux e Grafana

Projeto de estudos para instalar o Grafana em uma máquina virtual Ubuntu e aprender, na prática, como administrar um serviço Linux e acessá-lo pela rede.

Trabalho com monitoramento no NOC e estou construindo minha base em Linux, redes e observabilidade. A proposta é entender cada etapa, praticar os comandos e documentar os resultados.

## Estado atual

**Grafana instalado e página web acessível pelo navegador do Windows.**

- [x] Acessar a VM por SSH.
- [x] Consultar a versão do Ubuntu e o espaço em disco.
- [x] Preparar a chave pública e cadastrar o repositório oficial do Grafana.
- [x] Instalar o Grafana pelo APT.
- [x] Iniciar o serviço e acessar sua interface web.

As duas confirmações acima ainda não foram registradas neste README. A coleta de CPU, memória e disco ainda não está implementada.

## Ambiente

| Componente | Utilização |
| --- | --- |
| VirtualBox | Virtualização local |
| Ubuntu 22.04.5 LTS (Jammy) | Sistema operacional da VM |
| Terminal do Windows e SSH | Administração remota da VM |
| APT | Instalação do Grafana a partir do repositório oficial |
| systemd / systemctl | Gerenciamento do serviço grafana-server |
| Grafana OSS | Interface de visualização |

O Grafana foi instalado diretamente no Ubuntu, via APT. Esta etapa não utiliza container.

## Etapas realizadas

1. Identifiquei o IP da VM com `ip address` e conectei pelo Windows usando SSH.
2. Consultei o disco com `df -h` e a distribuição com `lsb_release -a`.
3. Preparei as ferramentas de instalação e o diretório `/etc/apt/keyrings`.
4. Baixei a chave pública do Grafana para `/etc/apt/keyrings/grafana.asc` e configurei a permissão `644`.
5. Cadastrei o repositório em `/etc/apt/sources.list.d/grafana.list`.
6. Atualizei o catálogo do APT e instalei o pacote `grafana`.
7. Iniciei o serviço `grafana-server` e acessei a página pelo navegador do Windows.

Referência de instalação: [documentação oficial para Debian e Ubuntu](https://grafana.com/docs/grafana/latest/setup-grafana/installation/debian/).

## Acesso ao laboratório

Substitua `IP_DA_VM` pelo endereço atual da sua VM:

```bash
ssh daniel@IP_DA_VM
```

No navegador do computador:

```text
http://IP_DA_VM:3000
```

O endereço IP pode mudar. Não inclua o sufixo de rede, como `/24`, no comando SSH ou na URL.

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

## Próxima etapa: métricas da própria VM

| Ferramenta | Papel planejado |
| --- | --- |
| Node Exporter | Expor métricas do Linux, como CPU, memória e disco |
| Prometheus | Coletar periodicamente as métricas e armazenar seu histórico |
| Grafana | Consultar o Prometheus como fonte de dados e apresentar os gráficos |

O primeiro objetivo desta etapa será exibir métricas reais da VM e entender como os dados chegam ao painel. O histórico começará a ser formado depois que a coleta estiver funcionando.

## Método de estudo

Uso cursos da Alura e um laboratório prático para consolidar os fundamentos. A sequência de referência é Comunicação Web, Unix e Shell e Sistema Operacional Linux, com exercícios relacionados ao projeto.

Durante a prática, tento explicar ou montar o comando antes de consultar a resposta. Comandos novos são apresentados com sua finalidade e opções, e cada etapa é conferida antes de avançar.

## Evidências

Ainda serão adicionadas capturas da interface do Grafana, do status do serviço e do primeiro painel com métricas. Antes de publicar imagens, revisar para não incluir senhas, tokens ou dados do trabalho.

## Autor

Daniel Alves de Albuquerque de Souza — estudos em Linux e observabilidade, com foco em evolução para SRE.
