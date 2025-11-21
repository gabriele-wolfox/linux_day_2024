# Laboratorio "Primi passi con Kubernetes"

## Link utili dalla documentazione di Kubernetes

- [Kubernetes Doc](https://kubernetes.io/it/docs/home/)
- [Kubernetes Components](https://kubernetes.io/docs/concepts/overview/components/)
- [kubectl quick reference](https://kubernetes.io/docs/reference/kubectl/quick-reference/)

## Primi passi

In questa esercitazione di laboratorio prenderemo confidenza con Kubernetes tramite `kubectl`,
un command line tool usato da amministratori per controllare i cluster Kubernetes, interagendo con l'Api Server.

In particolare, eseguiremo passo dopo passo un 
[esempio della documentazione](https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/), 
relativo all'esecuzione di un'applicazione stateless su Kubernetes,
tramite l'applicazione del manifest di Deployment.

Useremo un ambiente sandbox (gratuito per la prima ora consecutiva), disponibile su
[killercoda](https://killercoda.com).

1. Facciamo login, tramite Github account o email, alla [sandbox](https://killercoda.com/playgrounds/scenario/kubernetes)
2. Esploriamo il manifest di Deployment, prima di applicarlo
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
  Come possiamo vedere leggendo il manifest, faremo il deploy di due repliche di pod con immagini di Nginx
  nel container.
  Non interessante ai fini dell'esercitazione, Nginx, pronunciato “engine-ex”, è un web server open source 
  che, a cominciare dal suo successo iniziale come server, è ora utilizzato anche come proxy inverso, 
  cache HTTP e bilanciatore di carico.
  
3. Applichiamo il manifest di Deployment:
```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/application/deployment.yaml
```
4. Esaminiamo il Deployment e i Pod:
```shell
kubectl describe deployment nginx-deployment
kubectl get deployment
kubectl get deployment nginx-deployment -o yaml
kubectl get pods
kubectl get pods -l app=nginx
```
Dal risultato dell'ultimo comando, otteniamo i nomi dei Pod.
Per ciascuno dei pod, possiamo invocare l'API Server ed esaminare lo YAML del Pod
```shell
kubectl get pod <pod_name> -o yaml
```
5. Nello step successivo a questo, i pod verranno ricreati poiché cambieremo la versione dell'immagine Nginx nei container.
Se vogliamo vedere come cambia lo stato dei Pod, possiamo metterci "in watch" in un'altra Tab. 
Apriamo una nuova Tab cliccando sul `+`accanto a `Tab1`. 
Dopo esserci spostati sulla `Tab2`, possiamo monitorare i pod esistenti eseguendo il solito comando `kubectl get pods`, 
ma con `-w`come opzione, che sta per "watch".
Il comando resta in ascolto e non ci ritorna la shell, finché non lo abortiamo,
ad esempio con `Ctrl + C`.
```shell
kubectl get pods -w
```
6. Torniamo nella `Tab1` e applichiamo un altro manifest per il Deployment.
   Poiché il nome del Deployment del nuovo manifest coincide con il nome del Deployment già creato,
   il nuovo manifest editerà quello esistente.
   In particolare, aggiornerà l'immagine di Nginx nel container.
```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/application/deployment-update.yaml
```
7. Se torniamo nella `Tab2`, vediamo Pod terminati e nuovi, appena creati.
L'output sarà simile al seguente:
```shell
controlplane $ kubectl get pods -w
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-d556bf558-866vr   1/1     Running   0          2m50s
nginx-deployment-d556bf558-p9nr8   1/1     Running   0          2m50s
nginx-deployment-7dbfbc79cf-dfpxt   0/1     Pending   0          0s
nginx-deployment-7dbfbc79cf-dfpxt   0/1     Pending   0          0s
nginx-deployment-7dbfbc79cf-dfpxt   0/1     ContainerCreating   0          0s
nginx-deployment-7dbfbc79cf-dfpxt   0/1     ContainerCreating   0          0s
nginx-deployment-7dbfbc79cf-dfpxt   1/1     Running             0          5s
nginx-deployment-d556bf558-p9nr8    1/1     Terminating         0          3m23s
nginx-deployment-7dbfbc79cf-xrb9t   0/1     Pending             0          0s
nginx-deployment-7dbfbc79cf-xrb9t   0/1     Pending             0          0s
nginx-deployment-7dbfbc79cf-xrb9t   0/1     ContainerCreating   0          0s
nginx-deployment-d556bf558-p9nr8    1/1     Terminating         0          3m23s
nginx-deployment-d556bf558-p9nr8    0/1     Completed           0          3m23s
nginx-deployment-7dbfbc79cf-xrb9t   0/1     ContainerCreating   0          1s
nginx-deployment-d556bf558-p9nr8    0/1     Completed           0          3m24s
nginx-deployment-d556bf558-p9nr8    0/1     Completed           0          3m24s
nginx-deployment-7dbfbc79cf-xrb9t   1/1     Running             0          6s
nginx-deployment-d556bf558-866vr    1/1     Terminating         0          3m29s
nginx-deployment-d556bf558-866vr    1/1     Terminating         0          3m30s
nginx-deployment-d556bf558-866vr    0/1     Completed           0          3m30s
nginx-deployment-d556bf558-866vr    0/1     Completed           0          3m30s
nginx-deployment-d556bf558-866vr    0/1     Completed           0          3m30s
```
8. Torniamo nella `Tab1` ed editiamo il numero di pod che vogliamo il Deployment crei,
   aumentando le repliche dei Pod da 2 a 3
```shell 
kubectl scale deployment nginx-deployment --replicas=3
```
9. Controlliamo che c'è un pod in più
```shell 
kubectl get pods
```
10. Prendiamo l'indirizzo IP di uno dei Pod di Nginx dalla colonna IP, eseguendo
```shell
kubectl get pods -o wide
```
11. Se eseguiamo
```shell
curl <IP>
``` 
e copiamo il risultato in un tool per visualizzare HTML (es. [OneCompiler](htpps://onecompiler.com/html)) cosa otteniamo?
12. Entriamo nel pod
```shell
kubectl exec -it <pod_name> -- bash
```
Il parametro `-i` serve per passare lo `stdin` al container.
Il parametro `-t` serve per dire che lo stdin è tty.
Un dispositivo terminale tty è un terminale virtuale o console virtuale, 
cioè la combinazione di tastiera e schermo come interfaccia per interagire con un elaboratore.
13. Editiamo `index.html`.
```shell
cd /usr/share/nginx/html/
echo "Linux Day" > index.html
```
14. Usciamo dal Pod, eseguendo 
```shell
exit
```
15. Eseguiamo di nuovo 
```shell
curl <IP>
``` 
Scegliamo come IP quello relativo al Pod che abbiamo invocato con `curl` in precedenza.
Puoi trovare il comando nella history della shell, andando indietro con la freccia in alto, 
poiché hai già eseguito il comando al punto 11
16. Cancelliamo il Deployment
```kubectl delete deployment nginx-deployment``` 
e verifichiamo non ci sono più Pod e Deployment.
Che comando usiamo?

## Suggerimenti e convenzioni
- Durante l'esercitazione, al posto di `kubectl` puoi sempre usare il suo alias `k`
- Puoi sempre chiedere, tramite `kubectl` di "spiegarti" come è fatta una sua risorsa
Es. `kubectl explain deployment`
- Nell'ambito dell'esercitazione, YAML e manifest sono sinonimi
