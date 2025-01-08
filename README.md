### Simulador do Caos - Kubernetes Deployment

Este repositório contém a configuração para o deployment do **Simulador do Caos**, uma aplicação configurada para ser executada em um cluster Kubernetes. A configuração inclui o **Deployment** e o **Service** necessários para gerenciar e expor a aplicação.

---

## 🛠 Estrutura do Projeto

O projeto está dividido em duas partes principais:

1. **Deployment**  
   Configura o número de réplicas do pod e define as **probes de liveness**, garantindo que o estado da aplicação seja monitorado continuamente.

2. **Service**  
   Exponibiliza o **Simulador do Caos** por meio de um NodePort.

---

## 📄 Arquivos

- **`livenessprobe.yaml`**  
  Contém a configuração para o Deployment e Service no Kubernetes.

---

### Passo a Passo

1. **Aplique o Deployment e Service**  
   Execute o comando abaixo para criar os recursos no cluster Kubernetes:

   ```bash
   kubectl apply -f livenessprobe.yaml
   ```

2. **Verifique o Status do Deployment**  
   Certifique-se de que o pod está rodando corretamente:

   ```bash
   kubectl get pods
   ```

3. **Acesse a Aplicação**  
   Use o NodePort definido (32000) para acessar a aplicação:

   ```
   http://<NODE_IP>:32000
   ```

---

## ⚙️ Configurações Detalhadas

### **Liveness Probe** - Monitorando a Saúde do Container

A **Liveness Probe** é essencial para garantir que o container da aplicação esteja funcionando corretamente. Caso o container apresente problemas (como travamento ou falta de resposta), o Kubernetes reiniciará automaticamente o container.

#### Tipos Suportados:
- **HTTP GET**: Verifica se um endpoint específico da aplicação está respondendo.  
  Exemplo: `GET /health` na porta `3000`.
  
- **Exec**: Executa comandos dentro do container para verificar se ele está funcional.  
  Exemplo: `cat /tmp/health`.

- **TCP Socket**: Verifica se a aplicação está ouvindo em uma porta TCP específica.  
  Exemplo: Porta `8080`.

#### Parâmetros Configurados:
- **`initialDelaySeconds: 3`**  
  Aguarda 3 segundos após o início do container antes de iniciar as verificações.

- **`periodSeconds: 10`**  
  Realiza a verificação a cada 10 segundos.

- **`failureThreshold: 3`**  
  O container será reiniciado após 3 falhas consecutivas.

- **`timeoutSeconds: 3`**  
  Se a verificação demorar mais de 3 segundos, será considerada uma falha.

#### Exemplo de Configuração:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 3000
  initialDelaySeconds: 3
  periodSeconds: 10
  failureThreshold: 3
  timeoutSeconds: 3
```

### Service

- **Tipo**: NodePort  
- **Portas**:
  - Porta externa: `32000`
  - Porta interna: `3000`

---

## 🌟 Por que a Liveness Probe é Importante?

- **Auto-recuperação**: Se a aplicação parar de responder, o Kubernetes automaticamente reinicia o container.
- **Manutenção da disponibilidade**: Garante que o serviço continue disponível para os usuários finais.
- **Monitoramento proativo**: Detecta problemas antes que eles causem impacto significativo.

---

