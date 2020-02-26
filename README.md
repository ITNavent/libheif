# libheif


libheif is an ISO/IEC 23008-12:2017 HEIF file format decoder and encoder.

HEIF is a new image file format employing HEVC (h.265) image coding for the
best compression ratios currently possible.

libheif makes use of [libde265](https://github.com/strukturag/libde265) for
the actual image decoding and x265 for encoding. Alternative codecs for, e.g., AVC and JPEG can be
provided as plugins. There is experimental code for an AV1 plugin (for AVIF format support) in the 'avif' branch.

## C

Libreria desarrollada en C para el manejo de imagenes heif/heic. 

Como adaptación a las necesidades se creo una clase encargada a la conversion de imagenes heif/heic a la extension deseada. La clase en cuestion es HEIFConverter.cc, con su respectivo Header. Esta clase implementa el metodo con el codigo correspondiente a JNI, para poder ser consumido en un programa en JAVA. El Header es autogenerado a partir de la clase en JAVA, compilando la misma con el siguiente comando:
```
javac -h . HEIFConverter.java
```

## Instalacion

Se debe poseer de distintas librerias para la utilización de este proyecto. 
Librerias a poseer:

##### Libde265

Si no se posee, se puede instalar con este comando:
```
sudo apt-get install libde265-dev
```

##### gdk-pixbuf-2.0

Si no se posee, se puede instalar con este comando:
```
sudo apt-get install libgdk-pixbuf2.0-dev
```

##### libpng
Si no se posee, se puede instalar con este comando:
```
sudo apt-get install libpng-dev
```

##### x265

Si no se posee, se puede seguir instalar con este comando: 
```
sudo apt-get install x265
``` 
o seguir [este Instructivo](http://www.linuxfromscratch.org/blfs/view/svn/multimedia/x265.html) para la descarga e instalación.

## Compilacion

Para generar la libreria "libheif.so", se debe pararse en la raiz del proyecto y ejecutar:
```
cmake .
make
```
Dentro de la carpeta libheif, estara generada la libreria deseada.

Se debe copiar el que tiene la extension en numeros .1.6.2.0

Al momento de realizar el CMake, se debe ver que cosas faltan instalar.
