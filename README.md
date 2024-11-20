# **Jenkins and SonarQube Setup Guide**

![Uploading pin01-01.jpg…]()


Este proyecto muestra cómo instalar y configurar Jenkins y SonarQube para crear un entorno DevOps funcional. A continuación, un resumen de los pasos principales:

---

## **1. Configuración de Jenkins**

### **Instalación**
1. **Instalar Java 17**: Necesario para ejecutar Jenkins.  
2. **Instalar Jenkins**: Usar los paquetes oficiales y habilitar el servicio.  

### **Configuración**
1. Configurar nodos: Añadir un agente (servidor secundario) para distribuir las tareas.  
2. Instalar plugins importantes: Maven, Java y GitHub.  
3. Configurar herramientas como Maven y JDK desde **Manage Jenkins > Global Tool Configuration**.  
4. Añadir credenciales de GitHub para que Jenkins pueda interactuar con tus repositorios.  

---

## **2. Configuración de SonarQube**

### **Instalación**
1. **Instalar PostgreSQL**: Base de datos necesaria para SonarQube.  
2. **Instalar Java 17**: Necesario para ejecutar SonarQube.  
3. **Optimizar recursos**: Ajustar configuraciones del sistema para que SonarQube funcione eficientemente.  

### **Configuración**
1. Configurar base de datos en el archivo `sonar.properties`.  
2. Crear un servicio para que SonarQube se inicie automáticamente.  
3. Verificar que SonarQube esté corriendo en `http://<IP>:9000`.  

---

## **3. Integración Jenkins-SonarQube**

1. **Instalar plugins**: Instalar los plugins de SonarQube en Jenkins.  
2. **Configurar servidor de SonarQube** en Jenkins con la URL y un token de autenticación.  
3. **Añadir SonarQube Scanner** como herramienta en Jenkins.  

---

## **¿Qué implementamos?**
- **Automatización**: Jenkins distribuye tareas y ejecuta pipelines.  
- **Calidad del código**: SonarQube analiza tu código y asegura que cumpla con estándares de calidad.  
- **Colaboración eficiente**: Ambas herramientas se integran para facilitar el desarrollo y la implementación.  

--


Video Link -- https://youtu.be/e42hIYkvxoQ
============================================================= 

## Install and Configure the Jenkins-Master & Jenkins-Agent
=============================================================

## Install Java
$ sudo apt update
$ sudo apt upgrade
$ sudo nano /etc/hostname
$ sudo init 6
$ sudo apt install openjdk-17-jre
$ java -version

## Install Jenkins

Refer--https://www.jenkins.io/doc/book/installing/linux/

$ curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
  echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

$ sudo apt-get update
$ sudo apt-get install jenkins

$ sudo systemctl enable jenkins       //Enable the Jenkins service to start at boot
$ sudo systemctl start jenkins        //Start Jenkins as a service
$ systemctl status jenkins


$ sudo nano /etc/ssh/sshd_config   //Habilitar PubkeyAuthentication yes y 
				   //Habilitar AuthorizedKeyFile (en ambos)

				// En Jenkins-Master genero una clave publica. ssh-keygen
			        // copio el contenido de la clave pública
				// En Jenkins-Agent en /.shh/authorized_keys pego la clave

$ sudo service sshd reload
$ ssh-keygen OR $ ssh-keygen -t ed25519
$ cd .ssh

                                // Dentro de Jenkins-Master, voy a Nodes, Built-in-Node, Configure, Number of Executors:0 
		                // Creo un nuevo Nodo llamado Jenkins-Agent,
			        // Number of Executors:2,
			        // Host: es Private IPv4adresses Jenkins-Agent (ambos estan en la misma vpc)
				// Agrego una credencial Global
					 Kind SSH Username with private key
					 ID: Jenkins-Agent
					 Username: ubuntu
					 Private Key: Enter directly // pego la clave privada.
				// Host Key file Verification Strategy: Non Verifying Verification Strategy.
					 
// Instalar Plugins necesarios: Maven Integration, Pipeline Maven Integration, Eclipse Temurin Installer.

// configurar Plugins.
en Tools
	Maven Instalation
		Maven 3
			install automatically 
				version: 3.9.4
en Tools
	JDK Instalation
		Java17
			install automatically
				add instaler
					install from adoptium.net
						jdk-version
							jdk-17.0.5+8
// En credentials (configuro las credenciales de Github)
	Username with password
	Agon13
		Access Token 
							// Acces Token en Github 
								account
									setting
										 developer setting
											 personal acces token
												  generate new acces-token

	ID: github

							//Agrego Jenkinsfile al repo de Github

=============================================================
## Install and Configure the SonarQube 
=============================================================
## Update Package Repository and Upgrade Packages
    $ sudo apt update
    $ sudo apt upgrade
## Add PostgresSQL repository
    $ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
    $ wget -qO- https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/pgdg.asc &>/dev/null
## Install PostgreSQL
    $ sudo apt update
    $ sudo apt-get -y install postgresql postgresql-contrib
    $ sudo systemctl enable postgresql
## Create Database for Sonarqube
    $ sudo passwd postgres
    $ su - postgres
    $ createuser sonar
    $ psql 
    $ ALTER USER sonar WITH ENCRYPTED password 'sonar';
    $ CREATE DATABASE sonarqube OWNER sonar;
    $ grant all privileges on DATABASE sonarqube to sonar;
    $ \q
    $ exit
## Add Adoptium repository
    $ sudo bash
    $ wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
    $ echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
 ## Install Java 17
    $ apt update
    $ apt install temurin-17-jdk
    $ update-alternatives --config java
    $ /usr/bin/java --version
    $ exit 
## Linux Kernel Tuning
   # Increase Limits
    $ sudo vim /etc/security/limits.conf
    //Paste the below values at the bottom of the file
    sonarqube   -   nofile   65536
    sonarqube   -   nproc    4096

    # Increase Mapped Memory Regions
    sudo vim /etc/sysctl.conf
    //Paste the below values at the bottom of the file
    vm.max_map_count = 262144

#### Sonarqube Installation ####
## Download and Extract
    $ sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.65466.zip
    $ sudo apt install unzip
    $ sudo unzip sonarqube-9.9.0.65466.zip -d /opt
    $ sudo mv /opt/sonarqube-9.9.0.65466 /opt/sonarqube
## Create user and set permissions
     $ sudo groupadd sonar
     $ sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar
     $ sudo chown sonar:sonar /opt/sonarqube -R
## Update Sonarqube properties with DB credentials
     $ sudo vim /opt/sonarqube/conf/sonar.properties
     //Find and replace the below values, you might need to add the sonar.jdbc.url
     sonar.jdbc.username=sonar
     sonar.jdbc.password=sonar
     sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
## Create service for Sonarqube
$ sudo vim /etc/systemd/system/sonar.service
//Paste the below into the file
     [Unit]
     Description=SonarQube service
     After=syslog.target network.target

     [Service]
     Type=forking

     ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
     ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

     User=sonar
     Group=sonar
     Restart=always

     LimitNOFILE=65536
     LimitNPROC=4096

     [Install]
     WantedBy=multi-user.target

## Start Sonarqube and Enable service
     $ sudo systemctl start sonar
     $ sudo systemctl enable sonar
     $ sudo systemctl status sonar
     
//agrego la credencial de sonar en Jenkins
//Instalo plugins: sonarqube scanner, sonnar quality gates, y quality gates.
//Manage Jenkins, en Sonarqube Instalation: sonarqube-server, private ipv-4:9000
	//Token: jenkins-sonarqube-token

//en Tools
	en SonarQube Scanner: sonarqube-scanner
		install automatically
			SonarQube Scanner 5.0.1:3006

## Watch log files and monitor for startup
     $ sudo tail -f /opt/sonarqube/logs/sonar.log
