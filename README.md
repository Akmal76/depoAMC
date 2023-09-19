# depoAMC

**Tugas 3 - Pemrograman Berbasis Platform - Kelas D**

> **depoAMC** adalah aplikasi pengelolaan penyimpanan berbagai macam bahan dan perlengkapan yang diperlukan untuk proyek konstruksi, renovasi, perbaikan, atau pembangunan properti. 

## Perbedaan antara form `GET` dan form `POST` dalam Django
*Form* adalah cara untuk mengambil data dari pengguna. Data tersebut bisa berupa teks maupun *file*. Terdapat dua metode dalam mengirimkan data dari *browser user* yaitu `GET` dan `POST`.

<p align="center"><img src="https://1.bp.blogspot.com/-4XVpHqLWDSg/X4PUs7P6dWI/AAAAAAAAjyk/sWQQJ4ztHtQUs8GhDnmOdjSecNuqT9EpwCLcBGAsYHQ/w1200-h630-p-k-no-nu/difference_between_get_and_post_method_http.png" width="500"></p>

| Perbedaan | `GET` | `POST` |
|:--:|--|--|
|**Tujuan**| Mengambil atau membaca data dari Django (*web server*). | Mengirim data ke Django (*web server*). |
|**Penggunaan**| Melakukan pencarian atau menampilkan data. | Membuat, memperbarui, atau menghapus data. |
|**Cara Kerja**| Ketika pengguna mengisi form GET dan mengirimkannya, data yang dimasukkan ke dalam form akan muncul di URL. | Ketika pengguna mengisi form POST dan mengirimkannya, data yang dimasukkan ke dalam form dikirim sebagai bagian dari permintaan HTTP ke server, tetapi tidak muncul di URL |
|**Kode Status**| HTTP 200 (OK) jika berhasil | HTTP 201 jika berhasil |

## Perbedaan Utama antara XML, JSON, dan HTML dalam Konteks Pengiriman Data
| Perbedaan | XML | JSON | HTML |
|--|--|--|--|
| Penggunaan |Digunakan dalam pertukaran data antara sistem yang berbeda dan perlu menggambarkan data yang kompleks dan terstruktur dengan baik | Digunakan dalam pengembangan aplikasi web karena mudah dibaca oleh manusia dan mudah digunakan oleh bahasa pemrograman modern | Digunakan untuk menampilkan konten web sehingga dapat diakses oleh *browser* web |
| Sintaks | Menggunakan *tag* `<>` | Menggunakan *key-value pairs* `{}`, `[]`, dan `:` |  Menggunakan *tag* `<>` |

HTML bukanlah format data seperti XML dan JSON melainkan sebuah bahasa *markup* untuk menampilkan data.

<p align="center"><img src="https://upload.wikimedia.org/wikipedia/commons/5/56/JSON_vs._XML.png"></p>

## JSON sebagai Pertukaran Data antara Aplikasi Web Modern
JSON sering digunakan dalam pertukaran data antara aplikasi web modern karena kelebihan berikut:
1. Format data dan sintaks ringan, ringkas, sederhana, dan mudah dibaca.
2. Cocok untuk berbagai jenis data.
3. Didukung oleh banyak bahasa pemrograman sehingga cocok untuk *development* aplikasi web.

## Implementasi *Data Delivery*
### Membuat input `form`
Untuk mendapatkan data baru yang ingin ditampilkan, maka dapat dibuat `form` untuk menerima input.

1. Pertama, membuat berkas `forms.py` pada direktori `main` dan tambahkan kode berikut ini.

```python
from django.forms import ModelForm
from main.models import Item

class ItemForm(ModelForm):
    class Meta:
        model = Item
        fields = ["name", "amount", "description"]
```

2. Kedua, ubahlah fungsi `show_main` pada `views.py` dengan kode berikut ini.
```python
def show_main(request):
    items = Item.objects.all()

    context = {
        'name': 'Akmal Ramadhan',
        'class': 'PBP - D',
        'items': items,
    }

    return render(request, "main.html", context)
```

### Menambahkan fungsi pada `views`
Kita bisa melihat atau mengembalikan data yang telah dimasukkan melalui `form`. 

#### Format HTML

1. Untuk menerima data, akan dibuat fungsi baru bernama `create_item` pada `views.py` seperti pada kode berikut ini.
```python
from main.forms import Item

def create_item(request):
    form = ItemForm(request.POST or None)

    if form.is_valid() and request.method == "POST":
        form.save()
        return HttpResponseRedirect(reverse('main:show_main'))

    context = {'form': form}
    return render(request, "create_item.html", context)
```

2. Membuat *template* baru untuk tampilan dalam menambahkan *item* baru dengan nama `create_item.html` pada direktori `main/templates`.
```python
{% extends 'base.html' %} 

{% block content %}
<h1>Add New Item</h1>

<form method="POST">
    {% csrf_token %}
    <table>
        {{ form.as_table }}
        <tr>
            <td></td>
            <td>
                <input type="submit" value="Add Item"/>
            </td>
        </tr>
    </table>
</form>

{% endblock %}
```

3. Tampilkan data *item* dalam bentuk tabel dan tambahkan tombol `Add New Item` pada `main.html`.
```python
    <table>
        <tr>
            <th>Name</th>
            <th>Amount</th>
            <th>Description</th>
        </tr>
        
        <p> Terdapat <b> {{ items.count }} material atau bahan </b> yang tersedia. </p>

        {% for item in items %}
            <tr>
                <td>{{item.name}}</td>
                <td>{{item.amount}}</td>
                <td>{{item.description}}</td>
            </tr>
        {% endfor %}
    </table>
    
    <br />
    
    <a href="{% url 'main:create_item' %}">
        <button>
            Add New Item
        </button>
    </a>
```

---

Untuk format XML dan JSON, saya akan menambahkan *import* `HttpResponse` dan `serializers` pada `views.py` di folder `main`.

#### Format XML
Tambahkan fungsi `show_xml` yang me-*return* `HttpResponse` berisi data yang sudah di-*serialize* menjadi XML.

```python
def show_xml(request):
    data = Item.objects.all()
    return HttpResponse(serializers.serialize("xml", data), content_type="application/xml")
```

#### Format JSON
Tambahkan fungsi `show_json` yang me-*return* `HttpResponse` berisi data yang sudah di-*serialize* menjadi JSON.

```python
def show_json(request):
    data = Item.objects.all()
    return HttpResponse(serializers.serialize("json", data), content_type="application/json")
```

---

Untuk format XML dan JSON *by* ID, dalam pengambilan hasil *query* tambahkan *filter* dengan ID tertentu saja.

#### Format XML *by* ID
```python
def show_xml_by_id(request, id):
    data = Item.objects.filter(pk=id)
    return HttpResponse(serializers.serialize("xml", data), content_type="application/xml")
```

#### Format JSON *by* ID
```python
def show_json_by_id(request, id):
    data = Item.objects.filter(pk=id)
    return HttpResponse(serializers.serialize("json", data), content_type="application/json")
```
---

### Membuat *routing* URL
Tambahkan kelima *path url* fungsi diatas ke dalam `urlpatterns` pada `urls.py` di folder `main`. Tidak lupa untuk meng-*import*-nya dari `views.py`.

```python
from django.urls import path
from main.views import show_main, create_item, show_xml, show_json, show_xml_by_id, show_json_by_id 

app_name = 'main'

urlpatterns = [
    path('', show_main, name='show_main'),
    path('create-item', create_item, name='create_item'),
    path('xml/', show_xml, name='show_xml'), 
    path('json/', show_json, name='show_json'),
    path('xml/<int:id>/', show_xml_by_id, name='show_xml_by_id'),
    path('json/<int:id>/', show_json_by_id, name='show_json_by_id'),  
]
```

Dengan begitu, input `form` sudah selesai dibuat dan siap digunakan. Jalankan *command* `python manage.py runserver` dan kunjungi <http://localhost:8000>.

## Postman *Screenshot*
Berikut adalah tangkapan layar hasil akses URL melalui Postman untuk tiap kelima URL.
1. HTML
![HTML](image/postman_html.png)
2. XML
![XML](image/postman_xml.png)
3. JSON
![JSON](image/postman_json.png)
4. XML *by* ID
![XML *by* ID](image/postman_xml_1.png)
5. JSON *by* ID
![JSON *by* ID](image/postman_json_1.png)

## Referensi
Java67: [Difference between GET and POST Request in HTTP and REST APIs](https://www.java67.com/2014/08/difference-between-post-and-get-request.html)
Wikimedia: [JSON vs XML.png](https://commons.wikimedia.org/wiki/File:JSON_vs._XML.png)
PBP Ganjil 23/24: [Tutorial 2](https://pbp-fasilkom-ui.github.io/ganjil-2024/docs/tutorial-2)

## Arsip
<details>
<summary> <b> Tugas 2 </b> </summary>

## 👷🪓🪚 **Laman** 🚜⛏️🦺🔨
https://depoamc.adaptable.app/main/

## **Implementasi Aplikasi**
* ### Membuat proyek Django
Pertama, saya membuat direktori dan menyiapkan *dependencies* pada `requirements.txt` untuk menyiapkan proyek Django.

Berikut adalah isi dari berkas `requirements.txt`.
```
django
gunicorn
whitenoise
psycopg2-binary
requests
urllib3
```
Install *dependencies* tersebut dengan perintah `pip install -r requirements.txt` pada *virtual environment*. Setelah itu, proyek dibuat dengan menjalankan perintah `django-admin startproject depoAMC .` dan mengunggahnya ke repositori GitHub baru.

* ### Membuat aplikasi `main`
Pada direktori `depoAMC`, aktifkan *virtual environment* dan membuat aplikasi baru bernama `main` dengan perintah `python manage.py startapp main`. Daftarkan `main` ke dalam proyek dengan menambahkan `'main'` pada variabel `INSTALLED_APPS` yang berada di berkas `settings.py`.
```python
INSTALLED_APPS = [
    ...,
    'main',
    ...
]
```

* ### Membuat model aplikasi `main`
Model adalah bagian yang berhubungan dengan _database_. Pada berkas `models.py` di direktori `main`, saya mendefinisikan model sederhana baru dengan kode berikut ini.
```python
from django.db import models

# Create your models here.
class Item(models.Model):
    name = models.CharField(max_length=255)
    amount = models.IntegerField()
    description = models.TextField()
```
Tidak lupa untuk melakukan migrasi model dengan perintah `python manage.py makemigrations` dan menerapkannya ke *database* lokal dengan perintah `python manage.py migrate`.

* ### Membuat dan Menghubungkan Fungsi `views.py` dengan Template
Akan dilakukan *rendering* tampilan HTML dengan menggunakan data yang diberikan. Pada berkas `views.py` tambahkan `import render` dan fungsi `show_main` untuk menampilkan halaman `main.html` dengan kode dibawah ini.
```python
from django.shortcuts import render

def show_main(request):
    context = {
        'name': 'Akmal Ramadhan',
        'class': 'PBP - D'
    }

    return render(request, "main.html", context)
```
Pada `main.html`, saya meletakkan variabel yang dapat digantikan oleh data yang telah diambil dari model seperti dibawah ini.
```python
<p><b>Nama: </b>{{ name }}</p>
<p><b>Kelas: </b>{{ class }}</p>
```

* ### Melakukan *routing* aplikasi `main`
Untuk mengatur URL pada aplikasi `main`, saya membuat berkas `urls.py` pada aplikasi `main` yang berisikan kode berikut ini.
```python
from django.urls import path
from main.views import show_main

app_name = 'main'

urlpatterns = [
    path('', show_main, name='show_main'),
]
```
Agar URL proyek (`depoAMC`) dapat mengimpor URL aplikasi (`main`)
, maka pada berkas `urls.py` di `depoAMC` saya tambahkan fungsi `include` dari `django.urls` dan menambahkan URL ke tampilan `main` di dalam variabel `urlpatterns`.
```python
...
from django.urls import path, include
...

urlpatterns = [
    ...
    path('main/', include('main.urls')),
    ...
]
```
Dengan begitu, saya dapat melihat halaman `main` dengan perintah `python manage.py runserver` di [http://localhost:8000/main/](http://localhost:8000/main/)

* ### Melakukan *deployment* ke Adaptable
Langkah terakhir, saya melakukan *deploy* ke Adaptable dengan memilih `Python App Template` sebagai *template deployment* dan `PostgreSQL` sebagai *database type* yang akan digunakan. Pilih versi Python dengan versi lokal dan masukan _command_ `python manage.py && gunicorn DepoAMC.wsgi` pada `Start Command`.

## **Bagan**
![](image/bagan.jpg)

Ketika *web browser* menerima permintaan HTTP aplikasi `main` dari pengguna, terjadi URL *mapping* oleh `urls.py`. Setelah _mapping_ selesai dan ditemukan, fungsi pada `views.py` dipanggil sesuai dengan permintaan URL nya. Setelah itu, HTTP *request* ini akan dikembalikan oleh *view* menjadi HTTP *response* berupa HTML *page*. Dalam pengembalian ini, `views.py` akan mengakses data yang dibutuhkan dari `models.py` dan data tersebut ditampilkan menggunakan template `main.html`.

## **Virtual Environment**
Virtual Environment digunakan untuk memisahkan *packages* dan *dependencies* yang berbeda antar proyek dalam satu perangkat yang sama. Misalkan ketika ada dua proyek yang menggunakan versi Python yang berbeda. Dengan _virtual environment_, versi yang berbeda ini tidak saling mempengaruhi kedua proyek satu sama lain.

Kita bisa saja membuat aplikasi web berbasis Django tanpa menggunakan *virtual environment*. Akan tetapi, dapat terjadi risiko konflik *dependencies* antar satu proyek dengan proyek lainnya sehingga proyek yang bangun akan menjadi kacau.

## **MVC, MVT, dan MVVM**
Django menggunakan pola arsitektur MVT (Model-View-Template). Terdapat pola-pola lain seperti MVC dan MVVM.

Model: Mengelola data.

View:  Menerima input dan menampilkan informasi kepada pengguna.

#### 1. MVC (Model-View-Controller)
![Sumber: GeeksforGeeks](https://media.geeksforgeeks.org/wp-content/uploads/20201002214740/MVCSchema.png)

Controller: Berinteraksi dengan menghubungkan Model dan View sebagai pengatur *app flow* dan pengelola permintaan pengguna.

#### 2. MVT (Model-View-Template)
![Sumber: javaTpoint](https://www.javatpoint.com/django/images/django-mvt-based-control-flow.png)

Template: Mengatur tampilan HTML dan menggunakan data dari Model.

#### 3. MVVM (Model-View-Viewmodel)
![Sumber: GeeksforGeeks](https://media.geeksforgeeks.org/wp-content/uploads/20201002215007/MVVMSchema.png)

Viewmodel: Mengelola interaksi dan penghubung antara Model dan View serta mengubah data dari Model ke format yang dapat ditampilkan oleh View.

Perbedaan ketiga pola ini yaitu:

|                                          MVC                                          |                                               MVT                                                |                                                                          MVVM                                                                           |
|:-------------------------------------------------------------------------------------:|:------------------------------------------------------------------------------------------------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------:|
|                            Input diterima oleh Controller                             |                                     Input diterima oleh View                                     |                                                                Input diterima oleh View                                                                 |
|                       View dan Controller berelasi many-to-many                       |                              View dan Template berelasi one-to-one                               |                                                         View dan Viewmodel berelasi one-to-many                                                         |
| View tidak memiliki referensi ke Controller (panah satu arah dari Controller ke View) | View menyimpan referensi ke Template dan Template bekerja jika dipicu dari View (panah dua arah) | View tidak memiliki referensi ke Model dan sebaliknya. Viewmodel lah yang bertugas menghubungkan View dan Model. Dari sinilah nama Viewmodel digunakan. |

## Referensi
GeeksforGeeks: [Difference Between MVC, MVP and MVVM Architecture Pattern in Android](https://www.geeksforgeeks.org/difference-between-mvc-mvp-and-mvvm-architecture-pattern-in-android/)

Tomy's Blog: [MVC, MVP and MVVM](https://tomyrhymond.wordpress.com/2011/09/16/mvc-mvp-and-mvvm/)

javaTpoint: [Django MVT](https://www.javatpoint.com/django-mvt)

</details>