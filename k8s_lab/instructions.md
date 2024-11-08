# Laboratorio "Primi passi con Kubernetes"

## Link utili dalla documentazione di Kubernetes

[Kubernetes Doc](https://kubernetes.io/it/docs/home/)
[kubectl quick reference](https://kubernetes.io/docs/reference/kubectl/quick-reference/)

## Primi passi

In questa esercitazione di laboratorio prenderemo confidenza con Kubernetes tramite `kubectl`,
un command line tool usato da amministratori per controllare i cluster Kubernetes.

In particolare, eseguiremo passo dopo passo un 
[esempio della documentazione](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/), 
relativo all'esecuzione di un'applicazione stateless su Kubernetes,
tramite l'applicazione del manifest di Deplyoment.

Useremo un ambiente sandbox (gratuito per la prima ora consecutiva), disponibile su
[killercoda](https://killercoda.com).

1. Facciamo login, tramite Github account o email, alla [sandox](https://killercoda.com/playgrounds/scenario/kubernetes)
2. Esploriamo il Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
 
```
3. Applichiamo il manifest di Deployment:
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/application/deployment.yaml`
4. Esaminiamo il Deployment e i Pod:
- `kubectl describe deployment nginx-deployment`
- `kubectl get deployment` 
- `kubectl get deployment nginx-deployment -o yaml`
- `kubectl get pods` 
- `kubectl get pods -l app=nginx`
- `kubectl get pod <pod_name> -o yaml`
5. Applichiamo un altro manifest per il Deployment. 
Poichè il nome del Deployment del nuovo manifest coincide con il nome del Deployment già creato,
il nuovo manifest editerà quello esistente.
In particolare, aggiornerà l'immagine di Nginx nel container.
`kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/application/deployment-update.yaml`
6. Editiamo il numero di pod che vogliamo il Deployment crei, aumentando le repliche dei Pod da 2 a 3
`kubectl scale deployment nginx-deployment --replicas=3`
7. Controlliamo che c'è un pod in più
`kubectl get pods`
8. Vediamo la pagina `index.html` di uno dei Pod di Nginx, eseguendo
`curl <IP>` 
dove `<IP>` è uno degli indirizzi IP che vediamo nella colonna IP, eseguendo
`kubectl get pods -o wide`
9. Editiamo `index.html`, entrando nel pod
`k exec -it <pod_name> -- bash`
`cd /usr/share/nginx/html/`
`echo "Linux Day" > index.html`
10. Eseguiamo di nuovo `curl <IP>` scegliendo come IP quello relativo al Pod che abbiamo eseguito in precedenza
11. Cancelliamo il Deployment
`kubectl delete deployment nginx-deployment` e verifichiamo non ci sono più Pod e Deployment
