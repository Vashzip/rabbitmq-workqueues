# RabbitMQ Work Queues Atividade

## 📋 Sobre a Atividade

Este é um tutorial de RabbitMQ implementando o padrão "Work Queues" (também conhecido como Task Queues) - um sistema de distribuição de tarefas onde um produtor envia tarefas para uma fila e múltiplos workers (trabalhadores) competem para processar essas tarefas de forma balanceada.

# Seguindo o padrão de tutorial do próprio RabbitMQ Tutorial 2
# Tutorial realizado em Fedora Linux

## 🛠️ Ferramentas Utilizadas

### Software Obrigatório:
- **Python 3.13**
- **Conda Environment** Ambiente e pacotes
- **RabbitMQ Server** Servidor de mensageria RabbitMQ

### Bibliotecas Python:
- **pika** - Cliente Python para RabbitMQ

## 🚀 Passo a Passo Completo

### Configuração do Ambiente Conda

```bash
conda create -n rabbitmq python=3.13

conda activate rabbitmq

cd /home/vash/rabbitmq-work-queues
```

### Instalação do RabbitMQ

```bash
sudo dnf install rabbitmq-server

sudo systemctl start rabbitmq-server
sudo systemctl enable rabbitmq-server

sudo systemctl status rabbitmq-server
```

### Instalação do Cliente Python

```bash
conda install -c conda-forge pika
```

### Arquivos do Projeto

- **`new_task.py`** - Produtor que envia tarefas para a fila
- **`worker.py`** - Consumidor/Worker que processa as tarefas

Para parar as conexões quando necessário:

```bash
sudo systemctl stop rabbitmq-server
```

## 🔧 Como Executar

### 1. Ativar Ambiente e Verificar RabbitMQ

```bash
conda activate rabbitmq

sudo systemctl status rabbitmq-server
```

### 2. Iniciar Workers (Múltiplos Terminais)

**Terminal 1 - Worker 1:**
```bash
python worker.py
```

**Terminal 2 - Worker 2:**
```bash
python worker.py
```

**Saída esperada em cada worker:**
```
[*] Waiting for messages. To exit press CTRL+C
```

### 3. Enviar Tarefas (Producer)

Em outro terminal:

```bash
conda activate rabbitmq

python new_task.py "Hello World!"

python new_task.py "Uma tarefa longa....."

python new_task.py "Primeira tarefa."
python new_task.py "Segunda tarefa.."
python new_task.py "Terceira tarefa..."
```

**Saída esperada do produtor:**
```
[x] Sent 'Hello World!'
[x] Sent 'Uma tarefa longa.....'
```

### 4. Verificar Processamento nos Workers

Nos terminais dos workers, você verá as tarefas sendo distribuídas:

**Worker 1:**
```
[*] Waiting for messages. To exit press CTRL+C
[x] Received Hello World!
[x] Done
[x] Received Segunda tarefa..
[x] Done
```

**Worker 2:**
```
[*] Waiting for messages. To exit press CTRL+C
[x] Received Uma tarefa longa.....
[x] Done
[x] Received Terceira tarefa...
[x] Done
```

## 🔍 Como Funciona

### Conceitos do Work Queue:

- **Producer (new_task.py)**: Aplicação que envia tarefas
- **Task Queue**: Fila durável que armazena tarefas
- **Workers (worker.py)**: Múltiplos consumidores que competem pelas tarefas
- **Round-Robin Dispatch**: Distribuição circular entre workers disponíveis
- **Message Acknowledgment**: Confirmação de processamento
- **Fair Dispatch**: Distribuição baseada na disponibilidade do worker

### Fluxo de Tarefas:

```
Producer → Task Queue → Worker 1
                   ↘ → Worker 2
                   ↘ → Worker N
```

### Vantagens do Work Queue:

1. **Escalabilidade**: Adicione mais workers conforme necessário
2. **Confiabilidade**: Tarefas não são perdidas se um worker falhar
3. **Balanceamento**: Distribuição automática de carga
4. **Durabilidade**: Fila e mensagens persistem reinicializações

## 🧪 Testando Cenários

### Teste 1: Balanceamento de Carga
```bash
# Envie várias tarefas rapidamente
python new_task.py "Tarefa 1"
python new_task.py "Tarefa 2"  
python new_task.py "Tarefa 3"
python new_task.py "Tarefa 4"
```

### Teste 2: Tarefas com Diferentes Durações
```bash
python new_task.py "Rápida"
python new_task.py "Média.."
python new_task.py "Lenta....."
```

### Teste 3: Falha de Worker
1. Inicie uma tarefa longa
2. Pare o worker (Ctrl+C) durante o processamento
3. Observe a tarefa sendo redistribuída para outro worker