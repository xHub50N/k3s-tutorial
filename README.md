# Dokumentacja stworzenia własnego klastra Kubernetes opartego o K3S
<p style="text-align:center;">
[K3S webpage](https://k3s.io)
</p>

## 1. Przygotowanie maszyn
W moim przypadku będą to 2 maszyny wirtualne z systemem Ubuntu oraz z jedną kartą sieciową w trybie Bridge. Oraz podłączymy się do hosta z systemem windows aby móc z niego korzystać z poleceń kubectl oraz abyśmy mogli otworzyć dashboard!

Ustawiamy nazwy hostów odpowiednio master1, worker1 itd. w zależności od naszych potrzeb, najważniejsze aby dało się odróżnić jedną maszynę od drugiej.
Zmiana nazwy hosta odbywa się w pliku
`/etc/hostname` oraz w `/etc/hosts` -> `127.0.1.1 master1`

**WYŁĄCZAMY ZAPORĘ OGNIOWĄ** 

```ufw disable```

## 2. Instalacja serwera

Na maszynie z systemem ubutnu wykonujemy polecenie 

```curl -sfL https://get.k3s.io | sh - ```  

Pozwoli nam ono na ściągnięcie potrzebnych plików i klaster zostanie uruchomiony

![image](https://github.com/user-attachments/assets/2eeefd5a-6714-48cb-98fa-eddc87566685)

Wszystko zostało porawnie zainstalowane i nasz master istnieje w klastrze!

## 3. Konfiguracja Windowsa

Musimy pobrać plik kubectl ze strony [Releases](https://kubernetes.io/releases/download/#binaries)

Na systemie ubuntu na którym został zainstalowany klaster przechodzimy do folderu `/etc/rancher/k3s` i kopiuejmy zawartość pliku k3s.yaml do pliku na Windowsie: `C:\Users\{nazwa_usera}\.kube\config`

W pliku config edytujemy linijkę kodu `server: https://{adres maszyny z serwerem}:6443`

## 4. Dashboard

Aby pobrać dashboard do klastra kubernetes musimy wykonać polcenie na ubuntu:

```kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml```

Następnie musimy utworzyć uzytkownika na potrzeby usługi dashboardu

```sudo kubectl create serviceaccount dashboard-admin --namespace=kubernetes-dashboard```

```sudo kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:dashboard-admin```

Możemy sprawdzić czy nasz uzytkownik został utworzony

```sudo kubectl get serviceaccount dashboard-admin --namespace=kubernetes-dashboard```

![image](https://github.com/user-attachments/assets/c159918a-8d25-49a1-83a8-9a15eefd3e56)

Kolejnym krokiem jest utworzenie sekretu na potrzeby uwierzytelniania się z dashbaordem:

Tworzymy plik dashboard-admin-token.yaml a w nim umieszczamy zawartość

```
apiVersion: v1
kind: Secret
metadata:
  name: dashboard-admin-token-1234
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: dashboard-admin
type: kubernetes.io/service-account-token
```
I wykonujemy polecenie:

```kubectl apply -f dashboard-admin-token.yaml```

Aby odczytać nasz token musimy wykonać polecenie:

```kubectl describe secret dashboard-admin-token -n kubernetes-dashboard``` i kopiuejmy zawartość pozycji token:

Ostatnim krokiem jest przejście na windows i wykonanie polecenia:

```kubectl proxy``` 

i musimy przejść pod adres: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login

![image](https://github.com/user-attachments/assets/c0ed853f-a4af-43e3-8032-c7b440609b71)

Wklejamy nasz token i logujemy się 

![image](https://github.com/user-attachments/assets/4e3230d2-c618-4821-b765-4b5faed771f8)

Udało się!

## 5. Dodawanie workera do klastra

Na systemie mastera musimy odczytać i skopiować zawartość pliku: `/var/lib/rancher/k3s/server/node-token`

I na systemie workera wykonujemy polecenie: 

```curl -sfL https://get.k3s.io | K3S_URL=https://{adres mastera}:6443 K3S_TOKEN=token sh -```

![image](https://github.com/user-attachments/assets/6cbcbaa1-25f0-4942-a427-cae29b4e1c6b)

Udało się!
