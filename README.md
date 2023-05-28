<!-- basic catatan docker -->
-Docker Client untuk menjalankan perintah docker client agar mendaptkan respon dari sisi Server (Docker Daemon)
-Docker Daemon adalah server dari docker yg menyediakan tempat untuk membuat container dr aplikasi
-Docker Registry(Docker HUB) tempat untuk menaruh docker image
-Docker image mirip seperti installer app, dimana tanpa adanya docker images aplikasi tidak akan bisa jalan.
-Docker Container hasil dari installer docker images atau bisa dibilang applikasi jadi yg sudah di install

note: 1 docker images bisa membuat dokcer container berkali2 dengan catatan bahwa nama docker containernya harus berbeda

<!-- mengenai docker container: -->
-untuk melihat semua docker container yg sedang berjalan atau tidak bisa menggunakan command "docker container ls -a"
-apabila hanya melihat yg berjalan saja tinggal hilangkan -a

step play with docker: 
1. buat images, build images dengan command "docker build [OPTIONS] PATH | URL | -"
step play with container: 
2. buat containernya dengan command: docker container create --name namacontainer namaimage:tag
3. jalankan containernya dnegan command: docker container start contanainerid/namaconatainer (pilih salah satu menjalankan via id atau nama containernya)
4. untuk stop container dngn command: docker container stop containerid/namacontainer
5. menghapus container command: docker container rm containerid/namacontainer

<!-- note untuk docker cointainer -->
apabila docker cointainer dibuat dari 1 image yg sama dan container yg dibuat dr container 1 & 2 memiliki port yg sama maka aplikasi akan jalan seperti normal, 
karena masing2 container sudah terisolasi dng cointainer lain. Dimana kita menjalankan cotainer yg berbeda


```diff

+ (CONTAINER LOG) => untuk melihat log dari docker container
!  untuk melihat log container menggunakan comman: docker container log containerid/namacointainer / apabila mau melihat secara realtime tambahkan -f setelah log

----------------------------------------------------------------
container exec: mengeksekusi container yg dipilih
dengan menggunakan command line : container exec -i -t namacontainer /bin/bash lalu cd ke /

-----------------------------------------------------------------
docker port forwading: di mana kita bisa menjalankan container dengan port yg kita inginkan kedalam docker host, karena apabila kita hanya membuat docker default yg akan diberikan adalah port bawaan dari aplikasi yg kita mau jalankan, apabila container yg sudah kita buat sudah terinstall dengan port defaultnya maka kita tidak bisa memforwad port tersebut dengan port yg kita mau menggunakan docker host.Jd, docker container yg sudha dijalankan/dibuat tidak akan bisa diubah portnya menggunakan port dari docker host.

! contoh command docker port forwarding :
1. create container baru: docker container create --name namacontainer --publish expose(port yg diingikna):port(bawaan dari appnya) namaimage:tag

------------------------------------------------------------------
+ ENVIRONMENT VARIABLE : berguna untuk mengubah konfirgurasi aplikasi secara dinamis tanpa mengunah kode aplikasinya 

untuk menambah environmment variable bisa meggunakan perinta:
! --env atau -e : docker container create --name namacontainer --env KEY="value" --env KEY2="VALUE" namaimage:tag
! key dari env harus sama dengan env yg ada di app

------------------------------------------------------------------
+ CONTAINER STATS: melihat progress penggunaan dari setiap container,
contoh command: 
! docker container stats

--------------------------------------------------------------------
+ CONTAINER RESOURCE LIMIT: 
1. saat membuat container maka secara bawaan akan menggunakan cpu dan memory yg akan diberikan kedocker begitupun linux dimana akan menggunaka semua cpu dan memory yg tersedia ke sistem host
2. dampak buruk dari hal tersebut akan membuat container lain akan melambat akibat ada salah satu container yg mengambil terlalu banyak penggunaan cpu dan memorynya
3, maka dari itu container resource limit menjadi solve problem untuk membuat setiap container memiliki limitasi pengguanaan cpu & memory

!cara menggunakan atau mengaturnya :
1. memory :dengan menambhahkan --memory jumlahmemorynya(100m) => ket memory b(byte), k(kilbyte), m(megabyte), g(gigabyte),
2. cpu: --cpus 1.5 => penggunaan maksimal dari aplikasinya 

----------------------------------------------------------------------
+ DOCKER BINDING: (TIDAK DIREKOMENDASIKAN)
1. sistem sharing file atau folder yg terdapat didalam host ke dalam container yg ada didocker,
2. dengan sistem sharing ini file yg terdapat di dalam container walaupun container sudah terhapus data yg sudah dibuat masih bisa digunakan karena kita sudah menyimpan data tersebut kedalam file/folder di host
3. apabila kita tidak menggunakan docker binding maka docker akan membuatnya secara otomatis
4. isi dalam --mount memiliki aturan sendiri: 
    a. parameter type: tipe mount, bind, volume
    b. parameter source: sistem lokasi file/folder host
    c. parameter destination: lokasi file/folder container
    d. readonly: file/folder dalam kontainer tidak bisa ditulis hanya bisa dibaca saja
-note parameter destination: harus disamakan yg ada di container 
!sebagai contoh kita akan menggunakan images dari mongo dengan command sebagai berikut: 
! docker container create --name mongodata --publish 27018:27017 --mount "type=bind,source=folder dilaptomu,destination=foldercontainer,readonly " --env MONGO_INITDB_ROOT_USERNAME=ilham 
! --env MONGO_INITDB_ROOT_PASSWORD=ilham namaimage:latest

---------------------------------------------------------------------------
+ DOCKER VOLUME
1. mirip dengan docker bind mount: bedanya docker volume memilki management volume dimana bisa membuat volume, melihat jumlah daftar volume dan menghapus volume=> mirip2 kaya mau buat image
2. volume ini dianggap seperti storage karena bisa menyimpan data, ,bedanya dengan bind mount, volume ini peynimpanan data dimanage langsung oleh docker,
-note: jadi secara default ketika kita membuat sebuah container maka volume akan terbuat secara otomatis dengan name yg autogenerate untuk cek data volume : docker volume ls
- kita anggap bahwa volume ini adalah sebuah hardisk kosong yg nantinya akan menjadi tempat penyimpanan container yg inging menggunakannya.
- apabila volume ingin dihapus pastikan volume tersebut tidak digunakan
+cara membuat docker volume: 
! command: docker volume create volumemongodb
+cara menghapus docker volume: 
! command: docker volume rm namavolume

+ INTEGRASI VOLUME DENGAN CONTAINER: 
1. Apabila container dihapus maka data akan aman tersimpan di dalam volume
!command docker container menggunakan storage volume: docker container create --name mongodata --publish 27019:27017 --mount "type=volume,source=volumemongo,destination=/data/db" --env MONGO_INITDB_ROOT_USERNAME=ilham --env MONGO_INITDB_ROOT_PASSWORD=ilham mongo:latest
! --env MONGO_INITDB_ROOT_PASSWORD=ilham mongo:

+ BACKUP CONTAINER STORAGE 
1. untu backup data container ada cara yg bisa digunakan yaitu: 
    a. membuat container baru untuk menjadi trigger saat backup data volume yg ingin di simpan, untuk image yg ingin digunakan apa saja karena ini sebagai trigger untuk backup data
    contoh: - stop container yg menggunakan volume data yg ingin di backup 
            - membuat container baru dengan command: docker container create --name backup --mount "type=bind,source=/home/fdn-tech/Documents/belajar/doker belajar/basic docker/backup,destination=/backup" --mount "type=volume,source=volumemongo,destination=/data/db" nginx:latest
            - lalu jalankan containernya: docker container start backup
            - lalu masuk ke folder docker /bin/bash: docker exec -i -t namacontainer /bin/bash
            - lalu masuk kedalam folder back up dan archive dengan command: tar cvf targetarchive sourcefolderdatanya (tar cvf /backup/backup.tar.gz /data/cb)
2. cara shortcut dengan menggunakan container yg sekali pakai bisa langsung dhapus, hanya sebagai trigger backup. pakai image ubuntu: 
    contoh command: -jangan lupa matikan terlebih dahulu container yg menggunakan volume yg mau di backup
                    - masih menggunakan command yg sama seperti di nomor 1. a, akan tetapi ada tambahan command sebagai berikut: 
                    docker container run --rf --name backup-lagi --mount "type=bind,source=/home/fdn-tech/Documents/belajar/doker belajar/basic docker/backup,destination=/backup" --mount "type=volume,source=volumemongo,destination=/data/db" ubuntu:latest tar cvf /backup/backup-lagi/tar.gz /data
                    - yg di tambahkan dalam command di atas adalah (run --rf serta command tambahan di belakang image:tag)


+ DOCKER NETWORK: 
1. Berguna untuk container saling berkomunikasi, macam2 network yg sering dipakai yaitu: bridge, host, none 
    - network bridge dimana ketika kita sudah membuat kategori a dan dipakai 2 container maka container tersebut bisa saling berkomunikasi
contoh membuat docker network: 
! docker networkd create --drive bridge namadriver (apabila tanpa memakai --drive maka otomatis bakal default menjadi bridge)
- network tidak bisa dihapus apabila ada container yg masih menggunakannya
container yg terdapat dalam 1 network yg sama bisa salig berkomunikasi, container bisa akses another container dnegan menyebutkan hostname atau nama containernya
! cara menambahkan network pada container : docker container create --name blabla --network namanetwork image:tag

```

