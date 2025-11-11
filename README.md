### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/es-es/free/students/). Al hacerlo usted contará con $100 USD para gastar durante 12 meses.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

<img width="1601" height="779" alt="image" src="https://github.com/user-attachments/assets/ed28a47a-8983-48f0-9c56-6f8963111360" />

<img width="1598" height="781" alt="image" src="https://github.com/user-attachments/assets/54f3ffc7-1584-42c6-82cc-e7c98608fc13" />


2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).

   <img width="674" height="98" alt="image" src="https://github.com/user-attachments/assets/d2a121ff-1f18-4c4b-8db8-5f6c9d1c80de" />

4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

    <img width="928" height="217" alt="image" src="https://github.com/user-attachments/assets/8f34aa37-e647-460a-9071-8e290c3ce151" />

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` node FibonacciApp.js`

7. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

<img width="1594" height="772" alt="image" src="https://github.com/user-attachments/assets/9f0d4def-b053-4060-941d-ff294476eaf3" />

<img width="776" height="84" alt="image" src="https://github.com/user-attachments/assets/70222173-04ab-43eb-be23-8440810904b0" />

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
    
    <img width="1594" height="273" alt="image" src="https://github.com/user-attachments/assets/6939eb7c-68a4-441a-a2e5-915ef1485568" />

    * 1010000

    <img width="1600" height="272" alt="image" src="https://github.com/user-attachments/assets/2596ee2a-d4ea-4aa8-a6e5-e707b0be2ea9" />

    * 1020000

    <img width="1587" height="341" alt="image" src="https://github.com/user-attachments/assets/e9ae5433-09b3-47e0-9115-b8310aa6bbc0" />

    * 1030000

    <img width="1592" height="335" alt="image" src="https://github.com/user-attachments/assets/5c798ba2-4271-4c14-97b6-1634a3c8c3db" />

    * 1040000

    <img width="1611" height="340" alt="image" src="https://github.com/user-attachments/assets/7f95683c-f670-4de0-ab46-c993b79a5d6f" />

    * 1050000

    <img width="1590" height="338" alt="image" src="https://github.com/user-attachments/assets/d4926f2d-ce31-4dbd-a991-a4b7764c6ba0" />

    * 1060000

    <img width="1597" height="242" alt="image" src="https://github.com/user-attachments/assets/eaf95678-b10d-4192-bfd2-22b68cea89d0" />

    * 1070000

    <img width="1589" height="340" alt="image" src="https://github.com/user-attachments/assets/ff1d1bbe-b9ae-45f1-9e95-f63d4bd90198" />

    * 1080000

    <img width="1594" height="333" alt="image" src="https://github.com/user-attachments/assets/53c22529-78ad-476b-b9c1-b5bd4ecb8af2" />

    * 1090000    

    <img width="1595" height="338" alt="image" src="https://github.com/user-attachments/assets/13bf0cb7-441a-4542-ae07-2d5fd8d187c4" />

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

<img width="1356" height="744" alt="image" src="https://github.com/user-attachments/assets/bc307f01-c780-411a-8d47-47e525c8ac2f" />

<img width="1591" height="722" alt="image" src="https://github.com/user-attachments/assets/51bf54f6-7f02-4c6c-ab62-85792755f4dd" />

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

    <img width="1215" height="313" alt="image" src="https://github.com/user-attachments/assets/99991070-4712-4d95-bfcb-a210246dde68" />

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

<img width="1575" height="752" alt="image" src="https://github.com/user-attachments/assets/4cbde662-f50b-48f0-8c20-743989ac8eaa" />

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
    * 1000000

    <img width="1595" height="338" alt="image" src="https://github.com/user-attachments/assets/e03538cd-5930-4482-8bc8-2889d4d7c919" />

    * 1010000

    <img width="1592" height="339" alt="image" src="https://github.com/user-attachments/assets/33ed4fa9-e198-4ada-8631-c6bbbf7c2076" />

    * 1020000

    <img width="1599" height="336" alt="image" src="https://github.com/user-attachments/assets/00e249ce-45a2-4d0d-85ef-0d0ae3a40749" />

    * 1030000

    <img width="1597" height="336" alt="image" src="https://github.com/user-attachments/assets/02ded07e-427b-4db5-9077-8f4548042c6e" />

    * 1040000

    <img width="1592" height="339" alt="image" src="https://github.com/user-attachments/assets/51f75af5-e814-4ab2-847d-317321db551d" />

    * 1050000

    <img width="1596" height="334" alt="image" src="https://github.com/user-attachments/assets/86fd97cc-d87a-4c7a-8d80-8ac3b028893c" />

    * 1060000

    <img width="1600" height="332" alt="image" src="https://github.com/user-attachments/assets/6d064aab-c053-4b5f-a5be-db098aa1d48a" />

    * 1070000

    <img width="1604" height="336" alt="image" src="https://github.com/user-attachments/assets/f76ab426-9b8e-4753-8ada-03562d7a5f86" />

    * 1080000

    <img width="1603" height="338" alt="image" src="https://github.com/user-attachments/assets/ed81dd2e-e122-48ce-98d4-5081e4839e66" />

    * 1090000

    <img width="1597" height="252" alt="image" src="https://github.com/user-attachments/assets/4dfbff16-6514-436c-ac0e-86458004c469" />

consumo de CPU para la VM

<img width="1339" height="732" alt="image" src="https://github.com/user-attachments/assets/f97cdbb6-9e82-4dcb-8819-a21c8ffb50ac" />

<img width="1589" height="720" alt="image" src="https://github.com/user-attachments/assets/2dbb4517-3071-421c-a909-e59061d384bc" />

Postman para simular una carga concurrente a nuestro sistema:

<img width="1140" height="283" alt="image" src="https://github.com/user-attachments/assets/bc4f6193-cbbd-45c5-b058-c2ec387bbf03" />

12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.

Para el escenario de ESCALAMIENTO VERTICAL, al momento de hacer las pruebas cambiando el "B1ls" por "B2ms"; se puede ver una mejora tanto en el tiempo de ejecución a la hora de calcular los números,
Como en la cantidad de casos que puede ejecutar sin que fallen en una cantidad moderada.

Esto nos indica, una mejora en la capacidad del sistema a la hora de una petición mayor de información y toma de recursos; también indicando la estabilidad del sistema para no fallar o mostrar
errores tal que "ECONNRESET"


13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?

<img width="1665" height="660" alt="image" src="https://github.com/user-attachments/assets/bff7ab9f-7a0b-489f-84f9-e506fbb76938" />

2. ¿Brevemente describa para qué sirve cada recurso?

   - VERTICAL-SCALABILITY (Máquina virtual):
      Es el servidor principal donde se ejecuta el sistema operativo y las aplicaciones del usuario.

   - VERTICAL-SCALABILITY-ip (Dirección IP pública):
      Permite acceder a la máquina virtual desde Internet, asignándole una dirección visible externamente.

   - VERTICAL-SCALABILITY-nsg (Grupo de seguridad de red):
      Actúa como un firewall que controla el tráfico entrante y saliente hacia la máquina virtual.

   - vertical-scalability865_z2 (Interfaz de red - NIC):
      Conecta la máquina virtual con la red virtual y gestiona su comunicación en la nube.

   - VERTICAL-SCALABILITY_key (Clave SSH):
      Proporciona autenticación segura mediante par de claves, evitando el uso de contraseñas.

   - VERTICAL-SCALABILITY_OsDisk_... (Disco del SO):
      Contiene el sistema operativo y los archivos esenciales para el funcionamiento de la VM.

   - vnet-eastus2 (Red virtual):
      Define un entorno de red privado donde se ubican los recursos, aislando y segmentando el tráfico.
     
3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

   - Cuando se ejecuta la aplicación con un comando como npm FibonacciApp.js dentro de una sesión SSH, esta corre directamente en el entorno de esa conexión remota.
Al cerrar la sesión SSH, todos los procesos asociados a ella (incluyendo la aplicación Node.js) se terminan automáticamente.
Por eso, al salir del servidor o cerrar la ventana del terminal, la aplicación deja de ejecutarse y el servicio se detiene.

   - Azure bloquea el tráfico entrante hacia las máquinas virtuales para protegerlas de accesos no autorizados.
Por esta razón, si una aplicación escucha en el puerto 3000, los usuarios externos no podrán acceder hasta que se cree una regla de puerto entrante (Inbound port rule) en el Grupo de Seguridad de Red (NSG).
Esta regla autoriza explícitamente el tráfico hacia ese puerto, permitiendo que la aplicación sea accesible desde Internet de manera controlada y segura.

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

| Valor probado | Tiempo Antes (s) | Tiempo Después (s) |
| ------------- | ---------------- | ------------------ |
| 1000000       | 22.88            | 17.68              |
| 1010000       | 23.21            | 17.96              |
| 1020000       | 24.15            | 18.10              |
| 1030000       | 24.67            | 18.59              |
| 1040000       | 24.94            | 19.18              |
| 1050000       | 25.05            | 19.61              |
| 1060000       | 25.22            | 20.15              |
| 1070000       | 25.41            | 20.38              |
| 1080000       | 25.72            | 20.97              |
| 1090000       | 26.14            | 21.52              |

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

<img width="1909" height="857" alt="image" src="https://github.com/user-attachments/assets/e6329d27-24fc-4da1-a43e-b5d1af36b57b" />

Esta función llega a mantener estos pícos de porcentajes altos de consumo de CPU, puesto que la applicación está hecha con un algotritmo iterativo lineal.
Esto no ayuda a calcular la fórmula de fibonacci, para cuando son numeros grandes, puesto que necesita hacer ciclos de muchas iteraciones que ya calculó previamente.
Por esta razón, los numeros grandes necesitan de una cantidad mayor de CPU y memoria a la hora de hacer los cálculos.
Si este tuviera un sistema de cáclulo recursivo, que guarde los números ya calculados, el coste de memoria y CPU sería menor a los actuales.

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    * Si hubo fallos documentelos y explique.
   
   Tabla 1 con recursos B1ls:

   <img width="1049" height="791" alt="image" src="https://github.com/user-attachments/assets/614ff229-4e44-4fc6-8573-99eba9b3ce7d" />

   Tabla 2 con recursos B2ms:

   <img width="1196" height="783" alt="image" src="https://github.com/user-attachments/assets/013307b9-3bc3-4f7a-a367-55d77fe10e10" />

   El fallo "ECONNRESET"; se debe a la falta de memoria que puede llegar a necesitar la máquina, a la hora de calcular los números de un tamaño grande.
   Si se acaba los recursos de memoria y de CPU que le fueron dados a la máquina, saltará este error informando que el número no se pudo calcular de la forma esperada.

   De esta forma tambien podemos notar la diferencia entre los 2 recursos, en el cual el primero falla en 4 ocaciones, mientras que el segundo en ninguna y en un tiempo menor.

7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

   
   | Característica           |       B1ls                             |       B2ms                                  |
   | ------------------------ | -------------------------------------- | ------------------------------------------- |
   | vCPU                     | 1 vCPU básico                          | 2 vCPU con mayor rendimiento                |
   | Memoria RAM              | 0.5 GB                                 | 8 GB                                        |
   | Disco de datos           | 2                                      | 4                                           |
   | Almacenamiento local     | 4 SCSI                                 | 16 SCSI                                     |
   | E/S por segundo          | 320                                    | 1920                                        |
   | Desempeño                | Muy limitado, solo tareas ligeras      | Adecuado para cálculos y cargas moderadas   |
   | Rendimiento sostenido    | Bajo, puede bajar si se exige          | Alto y estable bajo carga continua          |
   | Relación costo–beneficio | Muy económico pero un bajo desempeño   | Mayor costo pero también mejor desempeño    |
   | Costo por mes            | 3.80 USD                               | 60.74 USD                                   |

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

   - Aumentar el tamaño de la máquina virtual es una solución válida solo a corto plazo.
Al asignar más CPU y memoria, la aplicación obtiene un mejor rendimiento y tiempos de respuesta más rápidos, como se observó al pasar de una B1ls a una B2ms, donde la FibonacciApp pudo procesar valores más altos de la secuencia de manera más estable.
No obstante, esta estrategia se basa únicamente en aumentar recursos físicos y no en mejorar la eficiencia del algoritmo.
Por ello, a medida que crece la carga o la complejidad del cálculo, el escalado vertical alcanza un límite físico y económico, dejando de ser una solución eficiente a largo plazo.

   - Al cambiar el tamaño de la VM, se asignan nuevos recursos de hardware (más núcleos, memoria, etc.), lo que permite que la FibonacciApp se ejecute más rápido y con mayor estabilidad.
Sin embargo, el cambio de tamaño no modifica el funcionamiento interno del programa: la aplicación sigue siendo monolítica y dependiente de una sola instancia.
En consecuencia, aunque se logra un aumento temporal del rendimiento, el problema de fondo persiste.

9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

   - Cuando se cambia el tamaño de una máquina virtual, Azure debe reasignar los recursos físicos (CPU, memoria y almacenamiento temporal) dentro del clúster donde está alojada.
Durante este proceso, la VM se detiene temporalmente para aplicar la nueva configuración, lo que puede generar una breve interrupción del servicio.
Una vez completado el cambio, la máquina se reinicia con los nuevos recursos asignados y vuelve a estar disponible con mayor capacidad de procesamiento.

   - El cambio de tamaño puede provocar pérdida de sesiones activas y tiempos de inactividad si no se planifica adecuadamente.
Además, incrementa los costos operativos, ya que los tamaños más grandes consumen más recursos y tienen tarifas superiores.
Por otra parte, el escalado vertical tiene un límite físico y económico.
Llega un punto en el que ya no es posible aumentar los recursos sin comprometer la rentabilidad o superar la capacidad disponible en la región.

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

Sí, hubo una mejora en los tiempos de respuesta.
La máquina virtual con mayor capacidad (B2ms) cuenta con más vCPUs y memoria, lo que permite ejecutar el cálculo de Fibonacci de manera más rápida y estable.
Aunque el consumo de CPU sigue siendo alto debido a la naturaleza intensiva del algoritmo, la nueva VM puede manejar la carga sin saturarse tan fácilmente como la anterior (B1ls).
Mas este proceso no ayuda en el paso del tiempo, cuando se le pidan al algoritmo una cantidad inmensa de peticioens y números grandes, este no va a poder responder.
Con lo cual es una buena solución a corto plazo para arreglar el tiempo de respuesta, pero solo para estos casos de números "pequeños".

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

Los siguientes datos se tomaron con la cantidad de recursos B1ls:

Tabla 1:

<img width="511" height="301" alt="image" src="https://github.com/user-attachments/assets/e4619da4-a961-44e4-8b3e-2f0fc3e6b88f" />

Tabla 2:

<img width="530" height="295" alt="image" src="https://github.com/user-attachments/assets/d55d4a4a-e1d0-4c3d-a3a9-60d4c82ddd68" />

Tabla 3:

<img width="522" height="290" alt="image" src="https://github.com/user-attachments/assets/d9c859db-3228-4eca-89ab-f22ca00cd6b5" />

Tabla 4:

<img width="527" height="298" alt="image" src="https://github.com/user-attachments/assets/e9706c28-0ebd-4bc6-8748-c9f630589148" />

El sistema no mejoró con 4 iteraciones en paralelo, dado que, de la forma en que está construido el algoritmo (de forma iterativa), este solo empeora cuando se le pide más recursos al mismo tiempo.
Dando por hecho que este algoritmo no tiene un grán escalamiento vertical a largo plazo y utilizandolo para diferentes iteraciones en paralelo.
Esto tambien sigue generando diferentes errores descritos previamente; con lo cual, no es buena idea pedirle a este algoritmo más iteraciones paralelas.

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
* ¿Cuál es el propósito del *Backend Pool*?
* ¿Cuál es el propósito del *Health Probe*?
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
* ¿Cuál es el propósito del *Network Security Group*?
* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la solución.




