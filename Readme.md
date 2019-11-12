![Texto alternativo a la imagen](https://i2.wp.com/blog.fincaraiz.com.co/wp-content/uploads/2019/07/logo_fincaraiz_w.png?fit=1024%2C178&ssl=1)

# fincaraiz_scrapping
Este proyecto se realiza con fines academicos, respondiendo a una practica de la maestria en ciencia de datos de la universidad abierta de cataluña, el propietario de estos datos es el portal web. 

## Objetivos

+ Aprender a aplicar los conocimientos adquiridos y su capacidad de resolución de problemas en entornos nuevos o poco conocidos dentro de contextos más amplios o multidisciplinarios.
+ Saber identificar los datos relevantes que su tratamiento aportan valor a una empresa y la identificación de nuevos proyectos analíticos.
+ Saber identificar los datos relevantes para realizar un proyecto analítico.
+ Capturar datos de diferentes fuentes de datos (tales como redes sociales, web de datos o repositorios) y mediante diferentes mecanismos (tales como queries, API y scraping).
+ Actuar con los principios éticos y legales relacionados con la manipulación de datos en función del ámbito de aplicación.
+ Desarrollar la capacidad de búsqueda, gestión y uso de la información y de los recursos.  

### ¿Qué página se ha seleccionado?
La pagina seleccionada ha sido : [finca raiz](https://www.fincaraiz.com.co/Vivienda/arriendo/), un portal de venta y arriendo que busca acercar a los ciudadanos diferentes opciones de vivienda, caracterizada por su facilidad de filtros y su aplicación móvil de búsqueda por posición geográfica. Actualmente tiene categorizados los inmuebles así:

+ Apartamento
+ Locales
+ Oficina
+ Casa
+ Bodegas
+ Apartaestudios
+ Habitaciones
+ Consultorios
+ Edificios
+ Lotes
+ Casas Campestres
+ Finca
+ Casa Lote
+ Parqueaderos
+ Cabañas

![Texto alternativo a la imagen](http://servitecmallorca.es/imagenes_fincaraiz/conociendo_sitio.JPG)


### ¿Por qué esta selección?

Se ha seleccionado porque se ve como un gran potencial de aplicación de técnicas de **Machine Learning**, gracias a sus reglas de negocio. Actualmente cualquier ciudadano del territorio Colombiano puede publicar su vivienda en este sitio web.

Por lo que, en una búsqueda de la estructura del negocio, se ha logrado evidenciar que al menos una de sus líneas de servicio es ofrecer pautas pagas por posicionamiento y etiqueta de destacado a cada uno de los inmuebles (ver imagen abajo). Refiriendome concretamente a que los clientes pagan por que su anuncio sea destacado por encima de los demás (al menos de manera visual por el contraste de colores).

![Texto alternativo a la imagen](http://servitecmallorca.es/imagenes_fincaraiz/negocio.JPG)

En esta idea, la propuesta fue __Extraer la mayor cantidad de datos con el fin de identificar, a través de modelos de *Machine Learning*, patrones en los anuncios con el objetivo de potencializar la propensión de compra de anuncios destacados__. Gracias a esto, se realizó el scrapp del portal, extrayendo los datos considerados relevantes (ciudad, barrio, precio, tipo de inmueble, entre otros) y los considerados variables clasificatorias _tiene pauta destacada, tiene pauta oportunidad, inmobiliaria_.

## Retos superados
Esta selección se debió al reto que  propiciaba y a la oportunidad de aprendizaje en el uso e implementación de técnicas de *scrapping* a través de Python. Es por esto que los retos más representativos para el proyecto fueron:

+ Comprender que la estructura de la paginación, resaltando que los elementos del paginado no cuentan con identificadores (etiqueta id) inicialmente, estos solo son creados cuando se da click en el item numero 2 de la paginación
+ iterar sobre las 2.774 paginas controlando tiempos entre cada iteración
+ Dar click a través de Selenium por cada pagina
+ Identificar la estructura del sitio (etiquetado)
+ Captura de las excepciones de selenium


## Trabajo Futuro

Sin duda, el reto que ha propiciado este sitio ha sido interesante, sin embargo y como es de esperar, queda la invitación a retomar el proyecto y realizar mejoras continuas. Entre estas, se proponen:

+ Extraer el contenido multimedia
+ Ingresar a cada uno de los inmuebles y extraer telefono de contacto, coordenadas en el mapa (a través de leaflet) y variables demográficas
+ Codificar en arquitectura POO


**Tamaño del Sitio**: Existen 83.193 registros (30 registros por fila) = 2.774 paginas
