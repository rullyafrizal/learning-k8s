# Catatan Belajar Kubernetes

## Arsitektur Kubernetes
![Kubernetes Architecture](/img/kubernetes-architecture.png)

### Kubernetes Master
- **kube-apiserver** bertugas sebagai API yang digunakan untuk berinteraksi dengan Kubernetes Cluster
- **etcd** bertugas sebagai database untuk menyimpan data Kubernetes Cluster
- **kube-scheduler** bertugas untuk memperhatikan aplikasi yang kita jalankan dan meminta Node untuk menjalankan aplikasi yang kita jalankan
- **kube-controller-manager** bertugas untuk melakukan kontrol terhadap Kubernetes Cluster
- **cloud-controller-manager** bertugas untuk melakukan kontrol terhadap interaksi dengan cloud provider

### Kubernetes Nodes
- **kubelet** berjalan di setiap Node dan bertugas untuk memastikan bahwa aplikasi kita berjalan di Node
- **kuber-proxy** berjalan di setiap Node dan bertugas sebagai proxy terhadap arus network yang masuk ke aplikasi kita dan sebagai load balancer juga
- **container-manager** berjalan di setiap Node dan bertugas sebagai container manager. Kubernetes mendukung beberapa container manager seperti Docker, containerd, rktlet, etc.

### Alur Kerja Kubernetes
![Alur Kerja Kubernetes](/img/alur-kerja-kubernetes.png)
#### Penjelasan
- Sebelum membuat configuration file, kita upload data aplikasi-aplikasi yang telah dibuild menjadi image ke image registry (ex: dockerhub, etc.)
- Kubernetes master akan mendistribusikan ke dalam Node dan memerintahkan kubelet untuk mengambil alih
- Kubelet akan memerintahkan docker atau container manager untuk pull image dari image registry, dan build container setelah itu

## Running Kubernetes Cluster Locally
- Install Using Minikube
- https://github.com/kubernetes/minikube
  
- Run minikube using VirtualBox
  ```
  minikube start --driver=virtualbox
  ```

#### Menginstall kubectl
https://kubernetes.io/id/docs/tasks/tools/install-kubectl/

## Kubernetes Node
- Node adalah worker machine di k8s, sebelumnya juga disebut minion
- Node bisa saja dalam bentuk VM (Cloud, etc.) atau server fisik
- Di dalam Node selalu terdapat beberapa kubelet, kube-proxy dan container manager

#### Melihat Semua Node
  ```
  kubectl get node
  ```
#### Melihat Detail Node
  ```
  kubectl describe node {nama_node}
  ```

## Kubernetes Pod
- Pod adalah unit terkecil yang bisa di-deploy di Kubernetes Cluster
- Pod berisi satu atau lebih container
- Secara sederhana Pod adalah aplikasi kita yang running di Kubernetes Cluster
- Satu Pod hanya bisa running di Satu Node saja

### Diagram Sederhana Pod
![Pod](/img/k8s-pod.png)

### Kenapa Butuh Pod?
- Kalau langsung menggunakan Container, Kubernetes hanya bisa satu spesifik container manager saja
- Sehingga dibuatlah satu abstraction layer bernama Pod
- Agar lebih memudahkan proses scaling

#### Melihat Semua Pod
  ```
  kubectl get pod
  ```

#### Melihat Detail Pod
  ```
  kubectl describe pod {nama_pod}
  ```

### Membuat Pod
- Template pod.yaml
  ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: pod-name
    spec:
      containers:
        - name: container-name
          image: image-name
          ports:
            - containerPort: 80
  ```
- **nama pod harus unique**

- Create command
  ```
  kubectl create -f {nama_file}
  ```

- Melihat pod yeng telah dibuat
  ```
  kubectl get pod
  ```
  ```
  kubectl get pod -o wide
  ```
  ```
  kubectl describe pod namepod
  ```

#### Mengakses Pod
  ```
  kubectl port-forward {nama_pod} portBinding:portPod
  ```
- example :
  `kubectl port-forward nginx 8889:80`



## Label
- Label adalah informasi tambahan yang ada di dalam resource Kubernetes
- Berguna untuk memberi tanda pada Pod, mengorganisir Pod
- Label tidak hanya bisa digunakan pada Pod, tapi pada semua resource di Kubernetes, seperti Replication Controller, Replica Set, Service, etc.

#### Show labels dari Pod
  ```
  kubectl get pods --show-labels
  ```

### Menambah atau Mengubah Label di Pod
**tidak disarankan, lebih bagus jika ditaruh di file config yaml**
- Menambah
  ```
  kubectl label pod {nama_pod} {key}={value}
  ```
- Mengubah
  ```
  kubectl label pod {nama_pod} {key}={value} --overwrite
  ```

#### Mencari Pod dengan label
- `kubectl get pods -l {key}` -> get Pod with label based on key
- `kubectl get pods -l {key}={value}` -> get Pod with label based on key and value
- `kubectl get pods -l '!{key}'` -> get Pod with label where not having this key
- `kubectl get pods -l {key}!={value}` -> get Pod with label where key not having this value
- `kubectl get pods -l '{key} in (value1, value2)'` -> get Pod with label with multiple value
- `kubectl get pods -l '{key} notin (value1, value2)'`

#### Mencari Pod dengan beberapa label
- `kubectl get pods -l key1,key2`
- `kubectl get pods -l key1=value,key2=value`
- `kubectl get pods -l key1,key2=value`


## Annotation
- Annotation mirip dengan Label, hanya tidak dapat difilter seperti label
- Annotation biasa digunakan untuk menambahkan informasi tambahan dalam ukuran besar
- Annotation bisa menampung informasi sampai 256kb

**Melihat Annotation harus lewat describe**
- `kubectl describe pod {nama_pod}`

#### Menambahkan Annotation ke Pod
- `kubectl annotate pod {nama_pod} {key}={value}`

#### Mengubah Annotation yang sudah ada
- `kubectl annotate pod {nama_pod} {key}={value} --overwrite`


## Namespace

### Kapan Menggunakan Namespace
- Ketika resources di Kubernetes sudah terlalu banyak
- Ketika butuh memisahkan resources untuk multi-tenant, team atau environment
- Nama resources bisa sama jika berada di namespace yang berbeda

#### Melihat Namespace
- `kubectl get namespaces`
- `kubectl get namespace`
- `kubectl get ns`

#### Melihat Pod di Namespace
- `kubectl get pod --namespace {nama_namespace}`
- `kubectl get pod -n {nama_namespace}`

#### Membuat Namespace
  ```
  kubectl create -f {nama_file}
  ```
#### Membuat Pod di suatu Namespace
  ```
  kubectl create -f {nama_file}.yaml --namespace {nama_namespace}
  ```
#### Menghapus Namespace
- **Semua resource (pod, dll.) yang ada di dalam namespace juga akan ikut terhapus**
  ```
  kubectl delete namespace {nama_namespace}
  ```

### Tentang Namespace
- Pod dengan nama yang sama boleh berjalan asalkan di Namespace yang berbeda
- Namespace bukanlah cara untuk mengisolasi resource, namespace hanya untuk grouping saja
- Walaupun berbeda namespace, pod akan tetap bisa saling berkomunikasi dengan pod lain di namespace yang berbeda


## Menghapus Pod
- `kubectl delete pod {nama_pod}`
- `kubectl delete pod {nama_pod1} {nama_pod2} ...`

### Menghapus Pod Menggunakan Label
  ```
  kubectl delete pod -l {key}={value}
  ```

### Menghapus Semua Pod dalam suatu Namespace
  ```
  kubectl delete pod --all --namespace {nama_namespace}
  ```

## Probe

### Liveness, Readiness, Startup Probe
- Kubelet menggunakan **Liveness Probe** untuk mengecek kapan perlu untuk restart pod (semacam health check). Jika saat dicek oleh liveness probe Pod tidak merespon, kubelet akan secara otomatis me-restart Pod.
- Kubelet menggunakan **Readiness Probe** untuk mengecek apakah Pod siap menerima traffic.
- Kubelet menggunakan **Startup Probe** untuk mengecek apakah Pod sudah berjalan, jika belum, maka kubelet tidak akan melakukan pengecekan liveness dan readiness
- Startup Probe cocok untuk Pod yang membutuhkan proses startup lama, ini dapa digunakan untuk memastikan Pod tidak mati oleh kubelet sebelum selesai berjalan dengan sempurna

### Mekanisme Pengecekan Probe
- HTTP Get (Cocok dipakai aplikasi web)
- TCP Socket (Alternatif dari HTTP Get, jika aplikasi hanya berbentuk socket saja)
- Command Exec (Alternatif dari kedua hal di atas)

### Konfigurasi Probe
- **initialDelaySeconds**, waktu delay setelah container jalan dan dilakukan pengecekan, default 0
- **periodSeconds**, seberapa sering pengecekan dilakukan, default 10
- **timeoutSeconds**, waktu timeout ketika pengecekan gagal, default 1
- **successThreshold**, ambang batas jumlah minimum dianggap sukses setelah berstatus failure, default 1
- **failureThreshold**, ambang batas jumlah minimum dianggap gagal, default 3

## Replication Controller
- Replication Controller bertugas untuk emmastikan bahwa Pod selalu berjalan
- Jika tiba-tiba Pod mati atau hilang, maka Replication Controller secara otomatis akan menjalankan Pod yang mati atau hilang tersebut
- Replication Controller biasanya ditugaskan untuk manage lebih dari 1 Pod
- Replication Controller akan memastikan jumlah Pod yang berjalan sejumlah yang telah ditentukan. Jika kurang, maka akan menambah Pod baru, jika lebih maka akan menghapus Pod yang sudah ada

### Diagram Replication Controller
![Replication Controller](/img/replication-controller.png)

### Isi Replication Controller
- Label Selector, sebagai penanda Pod
- Replica Count, jumlah Pod yang seharusnya berjalan
- Pod Template, template yang digunakan untuk menjalankan Pod

### Membuat Replication Controller
  ``` 
  kubectl create -f {nama_file}
  ```

#### Melihat Replication Controller
- `kubectl get replicationcontrollers`
- `kubectl get replicationcontroller`
- `kubectl get rc`

### Menghapus Replication Controller
- Saat kita menghapus Replication Controller, maka secara otomatis Pod yang berada pada label selector dari RC akan ikut terhapus

  ```
  kubectl delete rc {nama_rc}
  ```
- Apabila kita ingin menghapus Replication Controller tanpa menghapus Pod yang berada pada label selectornya, kita bisa tambahkan opsi `--cascade=false` atau `--cascade=orphan`
  ```
  kubectl delete rc {nama_rc} --cascade=false
  ```

## Replica Set
- Pada awalnya Replication Controller digunakan untuk menjaga jumlah replica Pod ddan me-reschedule ulang Pod yang mati atau hilang.
- Namun kini, telah dikenalkan resource baaru yang bernama Replica Set
- **Replica Set** adalah generasi terbaru dari Replication Controller, dan digunakan sebagai pengganti Replication Controller
- Penggunaan Replication Controller untuk saat ini sudah tidak direkomendasikan

### Replica Set Vs Replication Controller
- Replica Set memiliki kemampuan hampir mirip dengan Replication Controller
- Replica Set memiliki label selector yang lebih ekspresif dibandingkan Replication Controller yang hanya memiliki fitur label selector secara match

#### Melihat Replica Set
- `kubectl get rs`

