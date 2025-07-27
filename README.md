
# idnw9

Wazuh Ruleset for OpenStack Nova

Repositori ini berisi seperangkat decoder dan rules Wazuh yang dirancang khusus untuk memantau log dari layanan OpenStack Nova (nova-api, nova-compute, nova-scheduler). Ruleset ini bertujuan untuk meningkatkan visibilitas keamanan dan operasional pada infrastruktur OpenStack Anda dengan mengubah log mentah menjadi alert yang relevan dan dapat ditindaklanjuti.

1.Fitur Utama

    Deteksi Keamanan Kritis: Mendeteksi secara spesifik event penting seperti kegagalan autentikasi (401 Unauthorized).

    Pemantauan Operasional: Memberikan peringatan untuk masalah operasional seperti ketidaksesuaian status VM antara database dan hypervisor, atau kegagalan scheduling.

    Ekstraksi Konteks: Mem-parsing dan mengekstrak data penting dari log seperti instance_id, request_id, http_status, dan srcip untuk analisis mendalam.

    Pengurangan Noise: Mengabaikan log INFO yang bersifat operasional dan tidak kritikal untuk menjaga agar alert tetap relevan.

    Struktur Atomik: Dibangun menggunakan konsep atomic ruleset dan sibling decoders untuk kemudahan pengelolaan dan skalabilitas.

Instalasi

Untuk menggunakan ruleset ini, ikuti langkah-langkah berikut pada Wazuh Manager Anda:

1. Salin File Decoder

Salin konten dari file local_decoder.xml yang disediakan ke dalam file /var/ossec/etc/decoders/local_decoder.xml di Wazuh Manager. Jika file tersebut sudah ada, tambahkan konten ke dalamnya.
Bash

Buka atau buat file decoder lokal
nano /var/ossec/etc/decoders/local_decoder.xml
Salin dan tempel konten decoder dari repositori ini ke dalam file tersebut


2. Salin File Rules
Salin konten dari file local_rules.xml yang disediakan ke dalam file /var/ossec/etc/rules/local_rules.xml di Wazuh Manager. Jika file tersebut sudah ada, tambahkan konten ke dalamnya.
Bash
Buka atau buat file rules lokal
nano /var/ossec/etc/rules/local_rules.xml
Salin dan tempel konten rules dari repositori ini ke dalam file tersebut


3. Restart Wazuh Manager
Setelah menyimpan kedua file tersebut, restart layanan Wazuh Manager untuk menerapkan perubahan.
Bash
systemctl restart wazuh-manager


Konfigurasi Pengumpulan Log
Pastikan Wazuh Agent yang terpasang pada node OpenStack Anda dikonfigurasi untuk mengirim log Nova ke Manager. Tambahkan blok <localfile> berikut ke dalam file ossec.conf di agent Anda (biasanya di /var/ossec/etc/ossec.conf).
XML

<ossec_config>
  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/nova/*.log</location>
  </localfile>
</ossec_config>

Setelah menambahkan konfigurasi, restart Wazuh Agent.
Bash
systemctl restart wazuh-agent


Pengujian dan Validasi dengan wazuh-logtest

Anda dapat memvalidasi bahwa decoder dan rules bekerja dengan benar menggunakan utilitas wazuh-logtest pada Wazuh Manager.
Langkah-langkah Pengujian:

    Jalankan wazuh-logtest sebagai root.
    Bash

    /var/ossec/bin/wazuh-logtest

    Salin dan tempel salah satu baris log contoh di bawah ini, lalu tekan Enter.

    Periksa output pada Phase 2 (decoding) dan Phase 3 (filtering).

Contoh Log untuk Pengujian:

Tes 1: Kegagalan Autentikasi (Alert Level 10)

Input Log:
Code snippet

nova-api.log.1.2017-05-16_13:53:08 2017-05-16 06:25:02.867 25746 ERROR keystonemiddleware.auth_token [req-1cc7d50c-25a2-46b0-a668-9c00f589160c 113d3a99c3da401fbd62cc2caa5b96d2 54fadb412c4e40cdbaed9335e4c35a9e - - -] Identity response: {"error": {"message": "The request you have made requires authentication.", "code": 401, "title": "Unauthorized"}}

Output yang Diharapkan:

    Phase 2: Decoder nova-api-log-detail akan mem-parsing log.

    Phase 3: Rule 100110 akan terpicu dengan level 10 dan deskripsi "Nova API: Kegagalan Autentikasi...".

Tes 2: Peringatan Sinkronisasi VM (Alert Level 7)

Input Log:
Code snippet

nova-compute.log.2017-05-14_21:27:09 2017-05-14 19:39:09.660 2931 WARNING nova.compute.manager [req-addc1839-2ed5-4778-b57e-5854eb7b8b09 - - - - -] While synchronizing instance power states, found 1 instances in the database and 0 instances on the hypervisor.

Output yang Diharapkan:

    Phase 2: Decoder nova-compute-log-detail akan mem-parsing log.

    Phase 3: Rule 100210 akan terpicu dengan level 7 dan deskripsi "Nova Compute: Ketidaksesuaian status instance...".

Tes 3: Log HTTP (Alert Level 0 - Diabaikan)

Input Log:
Code snippet

nova-api.log.2017-05-14_21:27:04 2017-05-14 19:39:01.445 25746 INFO nova.osapi_compute.wsgi.server [req-5a2050e7-b381-4ae9-92d2-8b08e9f9f4c0 113d3a99c3da401fbd62cc2caa5b96d2 54fadb412c4e40cdbaed9335e4c35a9e - - -] 10.11.10.1 "GET /v2/54fadb412c4e40cdbaed9335e4c35a9e/servers/detail HTTP/1.1" status: 200 len: 1583 time: 0.1919448

Output yang Diharapkan:

    Phase 2: Decoder spesifik nova-api-log-http akan mem-parsing log dan mengekstrak semua field HTTP.

    Phase 3: Rule 100101 akan terpicu dengan level 0, sehingga tidak ada alert yang dihasilkan (noise reduction).
