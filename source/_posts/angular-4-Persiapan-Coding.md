---
title: Angular 4 - Persiapan Coding
date: 2017-09-18 21:50:34
tags:
---

Kali ini saya akan membahas mengenai seri angular 4 dari mulai setup project angular 4 sampai dengan implementasi dan integrasi dengan springboot, berikut adalah list yang akan kita bahas dalam seri angular 4 ini

1.[Persiapan Coding Angular 4](/angular-4-Persiapan-Coding/)
2.[Components Dan Data binding](/angular-4-Components-dan-Data-Binding/)
3.Directive
4.Service dan Dependency Injection
5.Routing
6.Observables
7.Form
8.Http Request

<!--more-->

# Persiapan Coding Angular 4
Disini kita akan membahas starting project angular 4 dengan menggunakan Angular CLI, apa itu angular CLI ? , lebih lengkapnya bisa dibaca [disini](https://cli.angular.io/) sebelum menginstal angular cli , kita diharuskan untuk menginstall nodejs terlebih dahulu, gunakan versi nodejs yang paling baru yang bisa didapatkan [disini](https://nodejs.org/en/download/). Untuk langkah-langkah installasi nodejs bisa dibaca [disini](https://nodejs.org/en/download/package-manager/). Jika nodejs sudah terpasang selanjutnya kita menginstall angular cli, caranya sangat mudah cukup dengan menjalankan perintah berikut didalam terminal/consol 

	npm install -g @angular/cli

Lakukan perintah berikut untuk memeriksa apakah angular cli sudah terpsang
	
	ng --version

Jika sukses maka akan muncul keterangan seperti ini

![angular cli version](1.png)

Kemudian untuk membuat sebuah project angular dengan menggunakan angular cli, jalankan perintah berikut

	ng new belajar-angular

jika berhasil akan muncul seperti gambar dibawah ini

![angular cli generate project](2.png)

Untuk menjalankan cukup dengan masuk kedalam directory aplikasi, kemudian jalankan perintah berikut

	ng serve

jika berhasil maka akan muncul seperti ini

![angular cli run project](3.png)

aplikasi tersebut akan berjalan di localhost dengan port 4200

![angular cli akses web](4.png)

Struktur project yang dibuatkan olah angular cli adalah sebagai berikut

![angular cli struktur project](5.png)

IDE yang saya gunakan diatas adala Visual Studio Code bisa anda untuh [disini](https://code.visualstudio.com/)
Demikian cara untuk setup project angular, selanjutnya kita akan membahas tentang [Components Dan Data binding](/angular-4-Components-dan-Data-Binding/).