# Laboratorium – Programowanie Full-Stack w Chmurze Obliczeniowej  
# Zadanie nr 1 – Część obowiązkowa

W ramach zadania utworzono klaster Kubernetes w oparciu o Minikube z wtyczką sieciową CNI Calico oraz sterownikiem Docker. Klaster składa się z węzła głównego (master) oraz trzech węzłów roboczych A, B oraz C.

Następnie wdrożono następujące komponenty:

- Deployment frontend (obraz nginx, 3 repliki) uruchomiony w przestrzeni nazw frontend, przypięty do węzła A
- Deployment backend (obraz nginx, 1 replika) uruchomiony w przestrzeni nazw backend, działający na tym samym węźle co baza danych
- Pod my-sql (obraz mysql) uruchomiony w przestrzeni nazw backend, działający na tym samym węźle co backend

Utworzono usługi:

- svc-frontend typu NodePort w przestrzeni nazw frontend
- svc-backend typu ClusterIP w przestrzeni nazw backend
- svc-mysql typu ClusterIP w przestrzeni nazw backend

Opracowano polityki sieciowe (NetworkPolicy), które zapewniają że:

- Pody Deploymentu frontend w przestrzeni nazw frontend mogą łączyć się wyłącznie z Pod-ami Deploymentu backend w przestrzeni nazw backend na porcie 80/TCP
- Pody backend w przestrzeni nazw backend mogą łączyć się z Pod-em my-sql wyłącznie na porcie 3306/TCP
- Pody backend mogą również inicjować połączenia z Pod-ami frontend
- Cały pozostały ruch sieciowy pomiędzy Pod-ami jest zablokowany

Dodatkowo skonfigurowano ograniczenia zasobów dla przestrzeni nazw:

- namespace frontend:
  - maksymalna liczba Pod-ów: 10
  - dostępne CPU: 1000m
  - dostępna pamięć RAM: 1.5Gi
- namespace backend:
  - maksymalna liczba Pod-ów: 3
  - dostępne CPU: 1000m
  - dostępna pamięć RAM: 1.0Gi

Dla Deploymentu frontend skonfigurowano autoskaler HPA, który dynamicznie skaluje liczbę replik na podstawie użycia CPU, nie przekraczając ograniczeń ResourceQuota dla przestrzeni nazw frontend.

Pliki w repozytorium:

- 01-frontend-deploy.yaml – Deployment frontend
- 02-backend-deploy.yaml – Deployment backend
- 03-mysql-pod.yaml – Pod my-sql
- 04-frontend-svc-nodeport.yaml – Service svc-frontend
- 05-backend-svc-clusterip.yaml – Service svc-backend
- 06-mysql-svc-clusterip.yaml – Service svc-mysql
- 07-netpol-frontend.yaml – NetworkPolicy dla frontend
- 08-netpol-backend.yaml – NetworkPolicy dla backend
- 09-frontend-quota.yaml – ResourceQuota dla frontend
- 10-backend-quota.yaml – ResourceQuota dla backend
- 11-hpa-frontend.yaml – Horizontal Pod Autoscaler

# 1) Start klastra (Calico + 4 nody: master + A,B,C)

- minikube delete
- minikube start --driver=docker --container-runtime=docker --network-plugin=cni --cni=calico --nodes=4

# 2) Label węzłów A/B/C

- kubectl get nodes
- kubectl label node minikube-m02 role=node-a --overwrite
- kubectl label node minikube-m03 role=node-b --overwrite
- kubectl label node minikube-m04 role=node-c --overwrite

# 3) Wdrożenie zasobów

- kubectl apply -f 01-frontend-deploy.yaml
- kubectl apply -f 02-backend-deploy.yaml
- kubectl apply -f 03-mysql-pod.yaml
- kubectl apply -f 04-frontend-svc-nodeport.yaml
- kubectl apply -f 05-backend-svc-clusterip.yaml
- kubectl apply -f 06-mysql-svc-clusterip.yaml
- kubectl apply -f 07-netpol-frontend.yaml
- kubectl apply -f 08-netpol-backend.yaml
- kubectl apply -f 09-frontend-quota.yaml
- kubectl apply -f 10-backend-quota.yaml
- kubectl apply -f 11-hpa-frontend.yaml

# Test poprawności konfiguracji (część G)

W celu zweryfikowania poprawności konfiguracji oraz działania autoskalera HPA uruchomiono test obciążeniowy w przestrzeni nazw frontend.

- Uruchomiono Pod generujący ciągły ruch HTTP do usługi svc-frontend
- Obserwowano wzrost liczby replik Deploymentu frontend
- Zweryfikowano, że nie zostały przekroczone limity CPU, RAM oraz maksymalna liczba Pod-ów

Test potwierdził poprawne działanie konfiguracji NetworkPolicy, ResourceQuota oraz HPA.

# [Potwierdzenie działania na zrzutach ekranu zapisanych w katalogu "zrzuty_ekranu"]
