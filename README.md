# Documentação Completa - **Simulador do Caos** no Kubernetes com Liveness e Readiness Probes

Este repositório contém a configuração necessária para o deployment do **Simulador do Caos** em um cluster Kubernetes. A configuração inclui tanto o **Deployment** quanto o **Service** para garantir a execução e acessibilidade da aplicação, além de implementar as **probes de liveness e readiness** para monitoramento contínuo da saúde e prontidão do serviço.

---

## 🛠 Estrutura do Projeto

O projeto está dividido em duas partes principais:

1. **Deployment**  
   Configura o número de réplicas do pod e define as **probes de liveness e readiness**, garantindo que o estado da aplicação seja monitorado e que o tráfego seja corretamente roteado para o serviço apenas quando ele estiver pronto.

2. **Service**  
   Exponibiliza o **Simulador do Caos** externamente, utilizando um **NodePort** para permitir o acesso à aplicação.

---

## 📄 Arquivos

- **`deployment.yaml`**  
  Contém a configuração do Deployment e do Service no Kubernetes.

---

## Passo a Passo

1. **Aplique o Deployment e o Service**  
   Execute o comando abaixo para criar os recursos no cluster Kubernetes:

   ```bash
   kubectl apply -f deployment.yaml
   ```

2. **Verifique o Status do Deployment**  
   Verifique se o pod foi criado e está rodando corretamente:

   ```bash
   kubectl get pods
   ```

3. **Acesse a Aplicação**  
   Utilize o **NodePort** configurado (32000) para acessar a aplicação via navegador ou curl:

   ```
   http://<NODE_IP>:32000
   ```

---

## ⚙️ Configurações Detalhadas

### **Liveness Probe** - Monitorando a Saúde do Container

A **Liveness Probe** é fundamental para garantir que o container da aplicação esteja em funcionamento. Caso o container apresente problemas como travamentos ou falhas de resposta, o Kubernetes reiniciará automaticamente o container, evitando a degradação do serviço.

#### Tipos Suportados:
- **HTTP GET**: Verifica se um endpoint HTTP específico da aplicação está respondendo corretamente.  
  Exemplo: `GET /health` na porta `3000`.
  
- **Exec**: Executa comandos dentro do container para verificar sua funcionalidade.  
  Exemplo: `cat /tmp/health`.

- **TCP Socket**: Verifica se a aplicação está ouvindo em uma porta TCP específica.  
  Exemplo: Porta `8080`.

#### Parâmetros Configurados:
- **`initialDelaySeconds: 3`**  
  Aguarda 3 segundos após o início do container antes de realizar a primeira verificação de liveness.
  
- **`periodSeconds: 10`**  
  Aprobe será executada a cada 10 segundos após o início da verificação.

- **`failureThreshold: 3`**  
  O container será reiniciado após 3 falhas consecutivas.

- **`timeoutSeconds: 3`**  
  Se a verificação exceder 3 segundos sem resposta, será considerada uma falha.

#### Exemplo de Configuração (Liveness Probe):

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

### **Readiness Probe** - Monitorando a Prontidão do Container

A **Readiness Probe** é usada para determinar quando o container está pronto para receber tráfego. Isso garante que o Kubernetes só roteie requisições para o container quando ele estiver totalmente inicializado e pronto para processar as solicitações, evitando enviar tráfego para uma aplicação que ainda não está operacional.

#### Tipos Suportados:
- **HTTP GET**: Verifica se um endpoint HTTP específico da aplicação está pronto para receber requisições.  
  Exemplo: `GET /ready` na porta `3000`.
  
- **Exec**: Executa comandos dentro do container para verificar sua prontidão.  
  Exemplo: `cat /tmp/ready`.

- **TCP Socket**: Verifica se o container está ouvindo em uma porta TCP específica.  
  Exemplo: Porta `8080`.

#### Parâmetros Configurados:
- **`initialDelaySeconds: 5`**  
  Aguardará 5 segundos após o início do container antes de realizar a primeira verificação de prontidão.

- **`periodSeconds: 5`**  
  Realiza verificações a cada 5 segundos.

- **`failureThreshold: 3`**  
  Após 3 falhas consecutivas, o Kubernetes considera o container "não pronto" e não enviará tráfego para ele.

- **`timeoutSeconds: 3`**  
  Se a verificação demorar mais de 3 segundos, será considerada uma falha.

#### Exemplo de Configuração (Readiness Probe):

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 3000
    httpHeaders:
      - name: token
        value: qualquer
  initialDelaySeconds: 5
  periodSeconds: 5
  failureThreshold: 3
  timeoutSeconds: 3
```

---

### **Service** - Expondo a Aplicação

O **Service** do Kubernetes é configurado para expor a aplicação externamente através de um **NodePort**, permitindo acesso via porta 32000.

#### Configuração do Service:

- **Tipo**: NodePort  
- **Portas**:
  - Porta externa: `32000`
  - Porta interna: `3000`

Exemplo de configuração do Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: simulador-do-caos
spec:
  selector:
    app: simulador-do-caos
  type: NodePort
  ports:
    - port: 80
      targetPort: 3000
      nodePort: 32000
```

---

## 🌟 Importância das Probes

### **Por que a Liveness Probe é Importante?**
- **Auto-recuperação**: Se o container travar ou falhar, o Kubernetes automaticamente o reinicia para tentar restaurar o serviço.
- **Manutenção da disponibilidade**: Evita que containers "mortos" continuem consumindo recursos e respondendo a requisições, o que impactaria negativamente a disponibilidade.
- **Monitoramento proativo**: Detecta falhas em um estágio inicial, antes que impactem os usuários finais.

### **Por que a Readiness Probe é Importante?**
- **Prevenção de tráfego indevido**: Garante que o Kubernetes só envie tráfego para containers que estão realmente prontos para processar requisições, evitando falhas de serviço.
- **Ajustes dinâmicos de tráfego**: Permite que o Kubernetes controle quando um pod pode começar a receber requisições, sem afetar os usuários finais durante o processo de inicialização ou recuperação.

---

## Diferenças entre **Liveness** e **Readiness** Probe

| **Aspecto**         | **Liveness Probe**                                      | **Readiness Probe**                                        |
|---------------------|---------------------------------------------------------|------------------------------------------------------------|
| **Objetivo**        | Verificar se o container está vivo e funcional.        | Verificar se o container está pronto para receber tráfego. |
| **Efeito no Tráfego** | Se falhar, o Kubernetes reinicia o container.         | Se falhar, o tráfego não será direcionado para o container.|
| **Quando usar**      | Para detectar falhas, como travamentos ou deadlocks.   | Para garantir que o container não receba tráfego até estar pronto. |
| **Exemplo de Uso**   | Recuperação de containers que travaram ou ficaram "presos". | Evitar envio de tráfego para um container que ainda está carregando ou inicializando. |

---

## Conclusão

As **Liveness Probe** e **Readiness Probe** são componentes fundamentais para garantir a resiliência e a disponibilidade dos serviços em containers no Kubernetes. No caso do **Simulador do Caos**, essas probes asseguram que a aplicação esteja monitorada quanto à saúde e prontidão, além de permitir uma recuperação automática em caso de falha. Com as probes configuradas corretamente, o Kubernetes pode garantir que o tráfego só seja roteado para containers saudáveis e prontos, garantindo uma experiência mais estável e resiliente para os usuários.
