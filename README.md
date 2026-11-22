# IABD-5074 — Despliegue y Auditoría de un Clúster Apache Hadoop 3 sobre Docker

Repositorio oficial del laboratorio práctico de **Big Data Aplicado**, centrado en el despliegue contenerizado, auditoría de bloques, tolerancia a fallos y ejecución de cómputo distribuido MapReduce sobre YARN utilizando **Apache Hadoop 3**.

---

## 1. Ficha Resumen de la Arquitectura Desplegada

El clúster se ejecuta mediante Docker Compose utilizando la imagen oficial `apache/hadoop:3`, interconectando **6 contenedores** con roles desacoplados sobre una red puente aislada (`docker_hadoop-net`).

### Topología de Nodos y Servicios

| Contenedor | Rol en el Clúster | Capa | IP Interna (Red Docker) | Puertos Expuestos / Función |
| :--- | :--- | :--- | :--- | :--- |
| **`namenode`** | Maestro HDFS | Almacenamiento | `172.23.0.2` | `9870` (Web UI), `8020` (RPC) — Gestión de metadatos y espacio de nombres |
| **`datanode1`** | Trabajador HDFS | Almacenamiento | `172.23.0.4` | `9866` (interno) — Almacenamiento físico de bloques de datos |
| **`datanode2`** | Trabajador HDFS | Almacenamiento | `172.23.0.3` | `9866` (interno) — Almacenamiento físico de bloques de datos |
| **`datanode3`** | Trabajador HDFS | Almacenamiento | `172.23.0.5` | `9866` (interno) — Almacenamiento físico de bloques de datos |
| **`resourcemanager`** | Maestro YARN | Cómputo | `172.23.0.6` | `8088` (Web UI), `8032` (RPC) — Planificación y asignación de recursos |
| **`nodemanager`** | Trabajador YARN | Cómputo | Dinámica | Ejecución de contenedores MapReduce (`2048 MB` RAM, `2 vCores`) |

### Parámetros Clave de Configuración (`docker/config`)

* **Sistema de archivos por defecto (`fs.defaultFS`):** `hdfs://namenode:8020`
* **Factor de replicación (`dfs.replication`):** `2` réplicas por bloque distribuidas entre distintos nodos trabajadores.
* **Tamaño de bloque (`dfs.blocksize`):** `67108864` bytes (**64 MiB**), optimizado para observar fragmentación física en entorno de laboratorio.
* **Capacidad agregada HDFS:** `3.243.303.530.496` bytes (**2,95 TiB** totales; `1006,85 GiB` por DataNode).
* **Gestor de ejecución (`mapreduce.framework.name`):** `yarn` con servicio auxiliar `mapreduce_shuffle`.
* **Recursos YARN (`nodemanager`):** `2048 MB` de memoria máxima asignable y `2 vCores`.

---

## 2. Estructura del Repositorio

```text
IABD-5074-LabHadoop-Alvarez-Estevez/
├── docker/
│   ├── docker-compose.yml    # Definición declarativa de los 6 servicios y red aislada
│   └── config                # Variables de entorno para core-site, hdfs-site, mapred-site y yarn-site
├── evidencias/
│   ├── 01_namenode_ui_9870.png
│   ├── 01_namenode_ui_9870_2.png
│   ├── 02_hdfs_fsck_bloques.png
│   ├── 02_hdfs_fsck_bloques_2.png
│   ├── 03_simulacion_fallo_datanode2.png
│   └── 04_yarn_wordcount_8088.png
├── informe/
│   └── Informe_Tecnico_Hadoop_Alvarez_Estevez.pdf
├── .gitignore
└── README.md
```

---

## 3. Instrucciones de Arranque y Uso

### Requisitos previos
* **Docker Desktop** (o Docker Engine + Docker Compose v2) en ejecución.
* Al menos **4 GB de memoria RAM** disponibles para los contenedores.

### Pasos para levantar el clúster

1. **Clonar el repositorio y acceder al directorio de infraestructura:**
   ```bash
   git clone https://github.com/manuelalvarez26-maker/IABD-5074-LabHadoop-Alvarez-Estevez.git
   cd IABD-5074-LabHadoop-Alvarez-Estevez/docker
   ```

2. **Iniciar los seis servicios en segundo plano:**
   ```bash
   docker compose up -d
   ```

3. **Verificar que todos los contenedores permanecen activos (`Up`):**
   ```bash
   docker compose ps
   ```

4. **Acceder a las interfaces web de monitorización:**
   * **HDFS NameNode UI:** [http://localhost:9870](http://localhost:9870) (estado del almacenamiento y pestaña *Datanodes*).
   * **YARN ResourceManager UI:** [http://localhost:8088](http://localhost:8088) (monitorización de nodos de cómputo y trabajos MapReduce).

5. **Abrir una sesión interactiva en el nodo maestro para operar sobre HDFS / YARN:**
   ```bash
   docker exec -it namenode bash
   ```

### Detención y limpieza del entorno

* **Detener los contenedores conservando los volúmenes de datos:**
  ```bash
  docker compose down
  ```
* **Detener y eliminar completamente los contenedores, redes y volúmenes asociados:**
  ```bash
  docker compose down -v
  ```

---

## 4. Aclaraciones sobre las Evidencias y el Informe Técnico

* **División de capturas gráficas para mayor legibilidad:** Con el objetivo de mejorar la legibilidad de las capturas sin perder resolución, he dividido determinadas evidencias almacenadas en el directorio `evidencias/` en dos capturas consecutivas (identificadas mediante el sufijo `_2`).
* **Alcance ampliado del informe técnico:** En el documento PDF entregado dentro de la carpeta `informe/`, además de dar respuesta detallada y justificada a las **cuatro cuestiones técnicas** exigidas en el guion de la práctica, he incorporado algunas explicaciones extra  junto con evidencias gráficas adicionales para dejar constancia de mi comprensión del ejercicio.

---

## 5. Autoría

* **Autores:** Manuel Álvarez Estévez
* **Módulo:** Big Data Aplicado (`IABD-5074`)
* **Fecha:** 22/11/2026
