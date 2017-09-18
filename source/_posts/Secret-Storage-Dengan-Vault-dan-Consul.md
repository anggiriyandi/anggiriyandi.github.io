---
title: Secret Storage Dengan Vault dan Consul
date: 2017-09-14 23:59:42
tags:
---

Artikel ini akan membahas tentang proses management secret storege dengan menggunakan vault dan consul .

## Instalasi consul 

 Download consul versi terbaru disini [download consul](https://www.consul.io/downloads.html). Extract di suatu folder, contoh : /opt/consul_app setelah itu kita set environment variable untuk consul.

	CONSUL_HOME=/opt/consul_app
	export CONSUL_HOME
	export PATH=$PATH:$CONSUL_HOME

Kemudian cek apakah variable CONSUL_HOME sudah dikenali sistem

	echo $CONSUL_HOME

Selanjutnya cek apakah consul sudah terinstal dan di kenali oleh system dengan cara sebagai berikut :

	consul -version

<!-- more -->

## Konfigurasi dan menjadikan consul sebagai service
Buat folder dengan nama 'data' di /opt/consul_app kemudian, buat file consule.service di /etc/systemd/system

 {% codeblock line_number:false highlight:true %}
	[Unit]
	Description=consul agent
	Requires=network-online.target
	After=network-online.target

	[Service]
	Environment=GOMAXPROCS=2
	Restart=on-failure
	ExecStart=/opt/consul_app/consul agent $OPTIONS -server -bootstrap-expect 1 -data-dir /opt/consul_app/data -bind 127.0.0.1
	ExecReload=/bin/kill -HUP $MAINPID
	KillSignal=SIGINT

	[Install]
	WantedBy=multi-user.target
{% endcodeblock %}

parameter '-data-dir /opt/consul_app/data' artinya file penyimpanan untuk consul akan disimpan di folder -data-dir /opt/consul_app/data.
Selanjutnya cek apakah consul sudah dikenali sistem sebagai service

	service consul status

untuk menjalankan consul lakukan perintah berikut

	service consul start

## Instalasi Vault
Download vault versi terbaru disini [download consul](https://www.vaultproject.io/downloads.html). Extract di suatu folder, contoh : /opt/vault_app setelah itu kita set environment variable untuk vault.

	VAULT_HOME=/opt/vault_app
	VAULT_ADDR='http://127.0.0.1:8200'
	export VAULT_HOME
	export VAULT_ADDR
	export PATH=$PATH:$VAULT_HOME

Kemudian cek apakah variable CONSUL_HOME sudah dikenali sistem

	echo $VAULT_HOME

Selanjutnya cek apakah consul sudah terinstal dan di kenali oleh system dengan cara sebagai berikut :

	vault -version

## Konfigurasi dan menjadikan vault sebagai service
Selanjutnya kita akan membuat file konfigurasi untuk vault, dan mengarahkan storage vault ke dalam consul.
Buat file config.hcl di /opt/vault_app

	storage "consul" {
  	 address = "127.0.0.1:8500"
  	 path = "vault"
	}	

	listener "tcp" {
 	 address = "127.0.0.1:8200"
 	 tls_disable = 1
	}

kemudian jadikan vault sebagai service di sistem, buat file vault.service di /etc/systemd/system

{% codeblock line_number:false highlight:true %}
[Unit]
Description=vault server
Requires=network-online.target
After=network-online.target consul.service

[Service]
Restart=on-failure
ExecStart=/opt/vault_app/vault server $OPTIONS -config=/opt/vault_app/config.hcl

[Install]
WantedBy=multi-user.target
{% endcodeblock %}

Kemudian cek apakah vault sudah dikenali sistem sebagai service

	service vault status

Untuk menjalankan service vault jalankan perintah berikut

	service vault start

Setelah vault dan consul berhasil terinstall di sistem selanjutnya lakukan langkah-langkah berikut untuk memulai bekerja dengan vault

### Initializing Vault
langkah pertama yang harus dilakukan setelah instalasi vault adalah inisialisasi vault tersebut, proses ini cukup dilakukan sekali untuk mendapatkan token root token dan unseal key. 

	vault init

jika berhasil maka akan muncul output seperti berikut

{% codeblock line_number:false highlight:true %}
Unseal Key 1: 5oQHfM6/wyRVCFQXH8KTS/1Lt8lfPmUlgmiDc2jcQAAG
Unseal Key 2: JbXBVq8mw9V1ivwhnN0DEHq0M8T04BNW7NSxjP1+M8V2
Unseal Key 3: G4rG9acAo0nM4UZpl/etI6D/WbJIn56r4VGvWQ88PRWC
Unseal Key 4: QHkkW7XuRPswsshJhMgYEflEY+1tNN2hHAZlqxxfvbww
Unseal Key 5: 8ZMul0z4oZ/TvNWSi4OyinWgudT2ipYDajeokox55A64
Initial Root Token: 10909686-c8aa-4de9-0b91-9dcc3832b558

Vault initialized with 5 keys and a key threshold of 3. Please
securely distribute the above keys. When the vault is re-sealed,
restarted, or stopped, you must provide at least 3 of these keys
to unseal it again.

Vault does not store the master key. Without at least 3 keys,
your vault will remain permanently sealed
{% endcodeblock %}

Ada beberapa key yang berhasil digenerate olah vault

Unseal key
Adalah key yang digunakan untuk proses unseal, kita tidak bisa melakukan apapun jika status vault masih dalam keadaan seal (protected), olehkarena itu kita membutuhkan key tersebut untuk membuat vault menjadi unseal dan siap digunakan. Dari keterangan diatas dapat dapat kita ketahui bahwa kita hanya cukup menggunakan 3 dari 5 unseal key untuk merubah status vault menjadi unseal. Jika kehilangan key tersebut maka vault tidak bisa dugunakan kembali, dan harus melakukan proses inisialisasi ulang , yang artinya data-data yang sebelumnya telah kita simpan akan hilang.

Initial Root Token
Adalah key yang digunakan untuk proses autentikasi sebagai root.

### Unseal vault
Untuk melakukan proses unseal cukup dengan perintah 

	vault unseal

kemudian masukan unseal key yang telah kita dapatkan sebelumnya, jika berhasil maka akan muncul output seperti berikut

{% codeblock line_number:false highlight:true %}
Key (will be hidden):
Sealed: true
Key Shares: 5
Key Threshold: 3
Unseal Progress: 1
Unseal Nonce: 7bb63398-2aef-233d-585a-7ebee8d716f1
{% endcodeblock %}

lakukan proses tersebut sebanya 3 kali, jalankan perintah berikut untuk melihat status vault

	vault status

kemudian akan muncul seperti dibawh ini 

{% codeblock line_number:false highlight:true %}
Sealed: false
Key Shares: 5
Key Threshold: 3
Unseal Progress: 0
Unseal Nonce: 
Version: 0.7.3
Cluster Name: vault-cluster-dfd0b91e
Cluster ID: a11599d2-0887-b184-0c13-a3f7f56c2e88
{% endcodeblock %}

jika kita berhasil melakukan proses unseal sebanyak 3 kali , status vault akan berubah manjadi status seal akan berubah menjadi false

### Autentikasi / login
Untuk melakukan autentikasi sebagai root , lakukan perintah berikut

	vault auth 

masukan root token yang telah didapat sebelumnya, jika berhasil akan muncul seperti dibawah ini

	Successfully authenticated! You are now logged in.
	token: cad1c4ce-43ef-4218-1ff6-e2480cdb19ce
	token_duration: 0
	token_policies: [root]


### Menulis dan membaca nilai pada vault
Untuk menulis secret didalam vault cukup lakukan perintah berikut

vault write secret/belajar password=123

jika berhasil akan keluar output seperti berikut

	Success! Data written to: secret/belajar

printah diatas artinya kita menulis sebuah secret dengan key = password dan value = 123 didalam prefix secret/, secret yang tersimpan didalam vault bersifat 'key value' .
Untuk membaca secret yang telah kita simpan cukup lakukan perintah 

	vault read secret/belajar

jika berhasil akan muncul seperti dibawah ini

	Key             	Value
	---             	-----
	refresh_interval	768h0m0s
	password        	123

kita juga bisa menyimpan beberapa key didalam vault , contoh 

	vault write secret/belajar password=123 secret=456

	vault read secret/belajar

jika berhasil akan muncul seperti dibawah ini

	Key             	Value
	---             	-----
	refresh_interval	768h0m0s
	password        	123
	secret          	456

Untuk menyimpan beberapa value disarankan menggunakan format json, vault sendiri sudah mendukung penyimpanan secret dengan menggunakan format json. Sebagai contoh kita akan membuat file json dengan nama secret.json di folder /tmp yang berisi

{% codeblock line_number:false highlight:true %}
{
  "data": {
     "excited": "yes",
     "value": "world"
  }
}

{% endcodeblock %}

	vault write secret/belajar @/tmp/secret.json

jika kita membaca secret tersebut dengan perintah 

	vault read secret/belajar

hasilnya akan seperti ini
	
	Key             	Value
	---             	-----
	refresh_interval	768h0m0s
	data            	map[excited:yes value:world]


### Secret Backend
Sebelumnya kita sudah mencoba menulis dan membaca nilai secret pada vault didalam prefix secret/. Prefix tersebut mengarah kepada backend yang akan kita gunakan, secara default prefix secret/ mengarah kepada backend yang dinamalan generic, generic backend menulis dan membaca raw data ke dalam backend storage. Selain generic backend ada beberapa backend yang di sediakan oleh vault, lebih lengkapnya ada [disini](https://www.vaultproject.io/docs/index.html)

Selain prefix secret/ kita juga bisa menambah prefix sendiri yang mengarah kepada secret backend yang kita inginkan. Contoh

	vault mount -path=rahasia generic

jika berhasil akan muncul keterangan berikut
	
	Successfully mounted 'generic' at 'rahasia'!

Selanjutnya untuk melihat prefix apa saja yang ada dan mengarah ke backend mana bisa melakukan perintah berikut

	vault mounts

jika berhasil akan muncul keterangan berikut

{% codeblock line_number:false highlight:true %}
Path        Type       Default TTL  Max TTL  Force No Cache  Replication Behavior  Description
cubbyhole/  cubbyhole  n/a          n/a      false           local                 per-token private secret storage
rahasia/    generic    system       system   false           replicated            
secret/     generic    system       system   false           replicated            generic secret storage
sys/        system     n/a          n/a      false           replicated            system endpoints used for control, policy and debugging
{% endcodeblock %}

Selain itu kita bisa menghapus path yang telah kita buat sebelumnya, yang berarti data-data yang telah kita simpan di path tersebut akan ikut terhapus, berikut adalah perintah untuk menghapus / unmount path yang telah kita buat sebelumnya

	vault unmount rahasia

### Vault Policy
Vault policy adalah sebuah otorisasi didalam vault yang membatasi akses dari masing-masing user / akses token. Policy tersebut disimpan dalam file konfigurasi .hcl (HashiCorp Configuration Language) untuk lebih lengkapnya bisa dibaca [disini](https://www.vaultproject.io/docs/concepts/policies.html). Untuk membuat policy, langkah pertama yang kita lakukan adalah membuat buat file misalnya configPolicy.hcl yang isinya sebagai berikut

{% codeblock line_number:false highlight:true %}

path "secret/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

path "secret/data" {
  capabilities = ["read"]
}


path "auth/token/lookup-self" {
  capabilities = ["read"]
}

{% endcodeblock %}

Selanjutnya kita simpan policy tersebut kedalam vault dengan perintah berikut

	vault policy-write agent-policy configPolicy.hcl

Perintah diatas artinya kita membuat policy dengan nama agent-policy yang aturannya diambil dari file configPolicy.hcl. Selanjutnya kita akan mambuat akses token dengan policy agent-policy. 

	vault token-create -policy="agent-policy"

jika berhasil akan keluar keterangan sebagai berikut

{% codeblock line_number:false highlight:true %}
Key            	Value
---            	-----
token          	d1ccfad6-375a-f3ef-8ca0-a5a633495832
token_accessor 	7efc53ec-6330-efb8-8f46-a83c7884bae4
token_duration 	768h0m0s
token_renewable	true
token_policies 	[agent-policy default]
{% endcodeblock %}

Kemudian kita login menggunakan akses token tersebut untuk memastikan akses token tersebut sudah benar-benar terikat dengan policy 'agent-policy'

	vault auth d1ccfad6-375a-f3ef-8ca0-a5a633495832

tulis sebuah nilai didalam path secret/data

	vault write secret/data secret=123

karena policy 'agent-policy' tidak membolehkan untuk menulis di path secret/data maka akan muncul keterangan berikut

	Error writing data to secret/data: Error making API request.

	URL: PUT http://127.0.0.1:8200/v1/secret/data
	Code: 403. Errors:

	* permission denied