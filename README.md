# Istio

<img src="./media/mundo-distribuido.png" />

## Benefícios do Istio

1. Gerenciamento de tráfego

    - Gateways (entrada e saída)
    - Load Balancing
    - Timeout
    - Política de Retry
    - Circuit Breaker
    - Fault Injection

2. Observabilidade

    - Métricas
    - Traces distribuídos
    - Logs

3. Segurança

    - Man-in-the-middle
    - mTLS
    - AAA (authentication, autorization e audit)

<img src="./media/dinamica-sidecar-proxy.png" />

<img src="./media/arquitetura-istio.png" />

-   Pilot: Controla todos os formatos de configurações;
-   Citadel: Trabalha com a parte de autenticação;
-   Galley: Traduz o que se faz para linguagem do Istio.

<img src="./media/monitoramento.png" />

## Instalação do k3d

O k3d proprociona um bind de portas mais simples de ser configurado.

Documentação: https://k3d.io

```bash
wget -q -O - https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```

## Criação do cluster e configuração do contexto

```bash
# --agents 2: são os nodes
k3d cluster create -p "8000:30000@loadbalancer" --agents 2

# Configuração do contexto
kubectl config use-context k3d-k3s-default
# Mensagem esperada: Switched to context "k3d-k3s-default".
```

## Instalar o Kubectl

Documentação: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

## Instalação do Istio

Documentação: https://istio.io/latest/docs/setup/getting-started/#download

```bash
# Baixa
curl -L https://istio.io/downloadIstio | sh -

# Move para pasta de /opt
sudo mv istio-1.18.2 /opt

# Configura a variável de ambiente
nano ~/.zshrc
export PATH="$PATH:/opt/istio-1.18.2/bin"
source ~/.zshrc

# Instala
istioctl install

# Testa
istioctl
```

## Verificações no Istio

```bash
# Verifica os namespaces e é possível observar o ns do istio
kubectl get ns
#NAME              STATUS   AGE
#default           Active   168m
#kube-system       Active   168m
#kube-public       Active   168m
#kube-node-lease   Active   168m
#istio-system      Active   5m6s

# Mostra o istiod e o ingressgateway rodando
kubectl get po -n istio-system
#NAME                                    READY   STATUS    RESTARTS   AGE
#istiod-977466b69-jqkgz                  1/1     Running   0          6m49s
#istio-ingressgateway-559fb9c9d9-txbhq   1/1     Running   0          6m37s

# Exibe os serviços do Istio
kubectl get svc -n istio-system
#NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                      AGE
#istiod                 ClusterIP      10.43.54.16     <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP        10m
#istio-ingressgateway   LoadBalancer   10.43.189.249   <pending>     15021:32037/TCP,80:30823/TCP,443:31745/TCP   9m59s
```

## K8s

```bash
# Roda o apply
kubectl apply -f deployment.yaml

# Verifica os pods
kubectl get pods

# Cria as labels para o istio proxy
kubectl label namespace default istio-injection=enabled

# Deleta o deploy do nginx
kubectl delete deploy nginx

# Roda o apply novamente
kubectl apply -f deployment.yaml

# Verifica o istio-proxy rodando
kubectl describe pod nginx-5d8974d7bd-nn7zv
# Created container istio-proxy
```
