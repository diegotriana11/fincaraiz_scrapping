# Lenguaje, Librerias y Codificacion

## Lenguajes
El lenguaje utilizado fue **Python - Jupyter Notebook**, con las siguientes liberias:

## Librerias
~~~python
import requests as req;
from bs4 import BeautifulSoup
import pandas as pd
import numpy as np
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.firefox.firefox_binary import FirefoxBinary
from selenium.common.exceptions import NoSuchElementException   
from selenium.common.exceptions import TimeoutException, WebDriverException
import time
~~~

## Codificacion  

### Apertura de url con Selenium   

Se hace uso de Selenium debido a que como proceso de prevencion de tecnicas de Scrapping, el portal ha ocultado los links de las 
paginas a través de funciones *onclick*, donde se evidencia que incialmente los elementos de pagination *a*, no cuentan con id; 
una vez se ha dado click en el primero y el segundo enlace de la paginacion, estos se transforman en elementos identificables por id.

~~~python

binary = FirefoxBinary(r'C:\Program Files (x86)\Mozilla Firefox\firefox.exe')
driver = webdriver.Firefox(firefox_binary=binary)
driver.get('https://www.fincaraiz.com.co/Vivienda/arriendos/')
~~~


### Proceso 1: Inicio con el scrap e identificacion de la paginacion

~~~python 
links = driver.find_elements_by_class_name('link-pag')
urls_registros = []
cantidad_cliks = 1
start = time.time()
print("Empezando proceso 1")

for link in links:   
    print("Procesando pagina: ", cantidad_cliks)
    if cantidad_cliks < 3:
        urls_registros.append(driver.current_url)   
        link.click()               
    else:     
#       funcion para recorrer los elementos por id  
        if existeEl("notification_panel"):        
            driver.execute_script("document.getElementById('notification_panel').remove()")            
            driver.find_element_by_id("lnkPage" + str(cantidad_cliks)).click()       
            urls_registros.append(driver.current_url)  
        else:       
            urls_registros.append(driver.current_url)
            print("Registro numero: ", cantidad_cliks ," guardado sin notificacion ")
            driver.find_element_by_id("lnkPage" + str(cantidad_cliks)).click()
        if cantidad_cliks > 40:
            break
    cantidad_cliks +=1
    
end = time.time()
print("Tiempo transcurrido ",end - start , " Segundos")
~~~

### Proceso 2: continuacion de la paginacion
~~~python
start = time.time()
print("Empezando proceso 2")
if cantidad_cliks > 7:
    data_urls = getExtraUrls(cantidad_cliks, urls_registros, 2774) #2773
    
end = time.time()
print("Tiempo transcurrido ",end - start)

~~~

### Proceso 3: concatenado de datos y exportacion a csv

~~~python
data=pd.DataFrame()
for url in data_urls.urls:
    data_scra = getRegistros(url)
#     print(data_scra)
    data=pd.concat([data,data_scra],sort=True)    
data = data.reset_index(drop=True)
data.to_csv('scrapp_fincaraiz.csv' ,sep=',')
~~~
### Funciones

La siguiente funcion se utiliza con el fin de conocer si un elemento de la pagina existe, ya que como mecanismo de proteccion, los 
desarrolladores de la pagina han colocado elementos que cubren la interaccion con elementos de click.  

~~~python
def existeEl(idelement):    
    driver.implicitly_wait(5)
    try:            
        driver.find_element_by_id(idelement)
        return True
    except WebDriverException as e:
        s = "%s" % e
        print("Got exception %s" % s)
        time.sleep(5)
        return False
~~~

A continuacion se presenta la funcion encargada de seguir con extraccion de datos una vez alcanzado el maximo inicial de paginacion.  

~~~python
def getExtraUrls(paginado, urls_registros, cantidad_paginas):
    
    for pag in np.arange(paginado, cantidad_paginas): 
        if existeEl("notification_panel"):        
            driver.execute_script("document.getElementById('notification_panel').remove()")
            driver.implicitly_wait(5)
            try:            
                driver.find_element_by_id("lnkPage" + str(pag)).click()
            except WebDriverException as e:
                s = "%s" % e
                print("Got exception %s" % s)
                time.sleep(5)                   
            urls_registros.append(driver.current_url)  
        else:       
            urls_registros.append(driver.current_url)
            print("Registro numero: ", pag ," guardado sin notificacion ")
            
            try:
                 driver.find_element_by_id("lnkPage" + str(pag)).click()
            except WebDriverException as e:
                s = "%s" % e
                print("Got exception %s" % s)
                time.sleep(5)     
        print("Se ejecuto la pagina: ",pag)  
    
    ## Creacion del dataframe con las urls a scrap
    urls_consulta = pd.DataFrame(urls_registros, columns=['urls'])
    urls_consulta = urls_consulta.drop_duplicates(subset=['urls'])
    return urls_consulta

~~~

Esta funcion se usa para extraer los registros de interes de cada pagina (*tipo_vivienda*,*ciudad_vivienda*, 
*barrio*, *dptos*, *precio*s, *urls*, *numero_habitaciones*, *metros*, *pauta_destacado* y *pauta_oportunidad*),
Inicialmente se recibe el parametro _webpage_, el cual se procesa a traves de *BeautifulSoup*, para finalmente extraer los elementos de la
pagina.

~~~python




def getRegistros(webpage):  
    headers = {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,\
    */*;q=0.8",
    "Accept-Encoding": "gzip, deflate, sdch, br",
    "Accept-Language": "en-US,en;q=0.8",
    "Cache-Control": "no-cache",
    "dnt": "1",
    "Pragma": "no-cache",
    "Upgrade-Insecure-Requests": "1",
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_3) AppleWebKit/5\
    37.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36"
    }
    page = req.get(webpage, headers=headers)
    soup = BeautifulSoup(page.text, 'lxml')
    print("Procesando pagina: ", soup.title.string)
    tipo_vivienda = []
    ciudad_vivienda = []
    barrio = []
    dptos = []
    precios = []
    urls = []
    numero_habitaciones = []
    metros =[]    
    pauta_destacado = []
    pauta_oportunidad = []
    viviendas = pd.DataFrame()
    
    try:    
        ul = soup.find_all('ul', class_="advert")
        for li in ul:
            
            ## Titulo de la publicacion, ciudad y barrio
            title = li.find('div', class_='span-title').h2.text.lower().split('-')
            if len(title) < 2:
                # titulo_registro = title[0].strip()
                tipo_inmueble = title[0].strip().split(' en ')[0]
                ciudad_inmueble = title[0].strip().split(' en ')[1]
                barrio_registro = "sin_informacion"
            else:
                # titulo_registro = title[0].strip()  
                tipo_inmueble = title[0].strip().split(' en ')[0]
                ciudad_inmueble = title[0].strip().split(' en ')[1]
                barrio_registro = title[1].strip()
            
            ## Link Personal a la pagina
            dpto  = li.find('div', class_="span-title").a
            url   = dpto.attrs.get('href')  

            ## Precio
            precio = li.find(attrs={'itemprop':'price'}).get('content')    

            ## Metros y Habitaciones
            info_vivienda = li.find_all(True, {"class":["surface", "li_advert"]})[0].text.strip().lower().split()  
            metros.append(info_vivienda[0])
            
            if len(info_vivienda) > 2:
                numero_habitaciones.append(info_vivienda[2])      
            else:
                numero_habitaciones.append("sin_informacion") 
                
            ## Etiquetas de Oportunidad
            pub_oportunidad = li.find('span', class_='etiqueta')   
            if pub_oportunidad is not None:                
                if len(pub_oportunidad.text) > 0:
                    pauta_oportunidad.append("1")
                else:
                    pauta_oportunidad.append("0")
                
            ## Pautas con la pagina
            pub_destacada = li.find('div', class_='icoov')   
            if pub_destacada is not None:                 
                if pub_destacada:
                    pauta_destacado.append("1")
                else:
                    pauta_destacado.append("0")           
            
            #### Extrayendo departamento
            subt  = dpto.find_all('div')
            indicador_dpto = 1
            for div in subt:
                if indicador_dpto == 2:
                    dpto_title = div.text.strip().lower()            
                indicador_dpto += 1
                
            ## Set Arreglos            
            tipo_vivienda.append(tipo_inmueble)
            ciudad_vivienda.append(ciudad_inmueble)
            barrio.append(barrio_registro)
            dptos.append(dpto_title)
            precios.append(precio)
            urls.append(webpage+url)
            

    
    except AttributeError:
        print("")
        
    ### Seteo del dataframe con los registros
    viviendas = pd.DataFrame()
    viviendas['tipo_vivienda'] = tipo_vivienda
    viviendas['ciudad_vivienda'] = ciudad_vivienda
    viviendas['departamento'] = dptos
    viviendas['barrio'] = barrio
    viviendas['numero_habitaciones'] = numero_habitaciones
    viviendas['metros'] = metros
    viviendas['precio'] = precios
    viviendas['url'] = urls
    viviendas['pauta_destacada'] = pauta_destacado
    viviendas['pauta_oportunidad'] = pauta_oportunidad
    
    
       

    return viviendas
    
~~~
