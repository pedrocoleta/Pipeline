def setConfigurationFile(){
	try{
		def props = readJSON file: 'configuration.json'
	}catch(Exception exp){
		credentialsFile = sh (script: "cat /opt/jenkins/credenciales_servidores/jenkins/id.txt", returnStdout: true)
		withCredentials([usernamePassword(credentialsId: "${credentialsFile}", usernameVariable: 'user', passwordVariable: 'pass')]) {
			sh label: 'Read configuration', script: "curl -u ${user}:${pass} -o configuration.json ${apiUrl}"
		}
	}
}
def setFicheroYaml(){
	def setMvn = caseMvn.find { it == "${mvnVersion}" }
	def mvnList = caseMvn.minus(setMvn)
	mvnList.add(0, setMvn)
	mvnOrder = mvnList.join('\n')

	def setJdk = caseJdk.find { it == "${jdkVersion}" }
	def jdkList = caseJdk.minus(setJdk)
	jdkList.add(0, setJdk)
	jdkOrder = jdkList.join('\n')

	def setSvrApp = caseSvrApp.find { it == "${svrApp}" }
	def svrAppList = caseSvrApp.minus(setSvrApp)
	svrAppList.add(0, setSvrApp)
	svrAppOrder = svrAppList.join('\n')

	def setEnviroments = caseEnv.find { it == "${enviroments}" }
	def enviromentList = caseEnv.minus(setEnviroments)
	enviromentList.add(0, setEnviroments)
	enviromentOrder = enviromentList.join('\n')

	def setPortManager = PortManager.find { it == "${portManager}" }
	def portManagerList = PortManager.minus(setPortManager)
	portManagerList.add(0, setPortManager)
	portManagerOrder = portManagerList.join('\n')

	def setContextPath = ContextPath.find { it == "${contextPath}" }
	def contextPathList = ContextPath.minus(setContextPath)
	contextPathList.add(0, setContextPath)
	contextPathOrder = contextPathList.join('\n')

	def jobConf = [
		'jobapp': "${JOB_NAME}",
		'urlgit': "${urlgit}",
		'mvnversion': "${mvnOrder}",
		'jdkversion': "${jdkOrder}",
		'enviroment': "${enviroments}",
		'enviroments': "${enviromentOrder}",
		'svrapp': "${svrAppOrder}",
		'server': "${params.server}",
		'portManager': "${portManagerOrder}",
		'contextPath': "${contextPathOrder}",
		'mvnOption': "${mvnOption}",
		'beforeBuildTask': "${beforeBuildTask}",
		'afterBuildTask': "${afterBuildTask}",
		'apiconfurl': "${apiUrl}"
	]

	writeYaml file: jobsFileConfApp, data: jobConf, overwrite: true
}

def deleteContext(String pathPom){
	def pom = readMavenPom file: pathPom
	try{
		def env = "${enviroments}".toString()
		def contexto = "${contextPath}".toString()
		def size = pom.profiles.size()

		for(i in 0..(size-1)) {
			def entorno = pom.profiles.get(i).properties.get('entorno')
			if(env == entorno) {
				value = i
			}
		}

		pom.profiles.get(value).properties.put('tomcat.contexto',contexto+'${project.build.finalName}')
		writeMavenPom model: pom, file: pathPom
	}catch(Exception exp){}
}

def setFilterPom(String pathPom){
	pom = readMavenPom file: pathPom
	entraFilter = false
	filterEliminarList = []
	try{
		filters = pom.getBuild().getFilters()
		if(filters != null){
			filters.each {
				if(it !=null && it!='${ficheroDeFiltro}'){
					entraFilter = true
					filterEliminarList.add(it)
				}
			}
		}
	}catch(Exception exp){}
	//Eliminar los filters
	filterEliminarList.each(){
		pom.getBuild().removeFilter(it)
	}
	if(entraFilter){
		pom.getBuild().addFilter('${ficheroDeFiltro}')
		writeMavenPom model: pom, file: pathPom
	}
}

def setPluginsPom(String pathPom){
	pom = readMavenPom file: pathPom
	//Obtener el pluginManagement de build
	entraPluginManagement = false
	pluginsEliminarList = []
	try{
		pluginManagement = pom.getBuild().getPluginManagement()
		if(pluginManagement != null){
			pluginManagement.getPlugins().each {
				entraPluginManagement = true
				if(it.artifactId == 'maven-site-plugin' || it.artifactId == 'maven-project-info-reports-plugin'){
					pluginsEliminarList.add(it)
				}
			}
		}
	}catch(Exception exp){}
	//Eliminar los plugins encontrados del pluginManagement
	pluginsEliminarList.each(){
		pom.getBuild().getPluginManagement().removePlugin(it)
	}
	
	//Obtener los plugins de build
	entraPlugin = false
	pluginsEliminarList = []
	try{
		plugins = pom.getBuild().getPlugins()
		plugins.each {
			entraPlugin = true
			if(it.artifactId == 'maven-site-plugin' || it.artifactId == 'maven-project-info-reports-plugin'){
				pluginsEliminarList.add(it)
			}
		}
	}catch(Exception exp){}
	//Eliminar los plugins encontrados de los plugins de build
	pluginsEliminarList.each(){
		pom.getBuild().removePlugin(it)
	}
	//Agregar los plugins deseados a build / pluginManagement
	org.apache.maven.model.Plugin plugin
	plugin = new Plugin()
	plugin.setGroupId('org.apache.maven.plugins')
	plugin.setArtifactId('maven-site-plugin')
	plugin.setVersion('3.7.1')
	if(entraPluginManagement && !entraPlugin){
		pom.getBuild().getPluginManagement().addPlugin(plugin)
	}else if (entraPlugin){
		pom.getBuild().addPlugin(plugin)
	}
	plugin = new Plugin()
	plugin.setGroupId('org.apache.maven.plugins')
	plugin.setArtifactId('maven-project-info-reports-plugin')
	plugin.setVersion('2.9')
	if(entraPluginManagement && !entraPlugin){
		pom.getBuild().getPluginManagement().addPlugin(plugin)
	}else if (entraPlugin){
		pom.getBuild().addPlugin(plugin)
	}

	//Obtener reporting del pom
	pluginsEliminarList = []
	reporting = pom.getReporting()
	if(reporting != null){
		reporting.getPlugins().each {
			pluginsEliminarList.add(it)
		}
		pluginsEliminarList.each(){
			pom.getReporting().removePlugin(it)
		}
	}
	
	//Obtener scm del pom
	scm = pom.getScm()
	if(scm != null){
		scm.setConnection("scm:svn:${urlSvn}${tagSvn}")
		scm.setDeveloperConnection("scm:svn:${urlSvn}${tagSvn}")
		scm.setUrl("${urlSvn}${tagSvn}")
	}
	
	//Escribir los cambios en el pom
	writeMavenPom model: pom, file: pathPom
}

node  {

// Recuperacion de configuracion
  stage('Run Jenkinsfiles') {    
    script {
	
			valorInfo = 'java 3.1.0 - Fecha Creación: 15/06/2021 - Fecha Modificación: 18/10/2021'
			textoInfo = 'Información acerca del pipeline'
			textoUrlGit = 'URL del repositorio GIT en el que se encuentra el código a usar'
			textoMaven = 'Versión de maven a utilizar en la compilación del código fuente'
			textoJDK = 'Versión de la máquina virtual de java (linux) a utilizar en la compilación del código fuente'
			textoEnviroments = 'Entorno sobre el que realizar la compilación y despliegue del artefacto'
			textoServer = 'Máquina física o virtual sobre la que se realizará el despliegue del artefacto. Se puede elegir más de una utilizando la tecla CTRL o SHIFT'
			textoSvrApp = 'Servidor de aplicaciones donde desplegar el artefacto generado.\n• tomcat → Apache Tomcat 5.x\n• tomcat6 → Apache Tomcat 6.x\n• tomcat7 → Apache Tomcat 7.x\n• tomcat8 → Apache Tomcat 8.x\n• tomcat9 → Apache Tomcat 9.x\n • wildfly → Wildfly X.X'
			textoPuerto = 'Puerto del Tomcat Manager donde desplegar el artefacto generado'
			textoContexto = 'URL virtual relativa para despliegue del artefacto'
			textoMvnOption = 'Parámetros adicionales en la instrucción de maven. La instrucción por defecto es: mvn clean -Dmaven.test.skip=true package. En el caso de que se necesite más de un parámetro en la instrucción, éstos se pondrán separados por espacios\n Ejemplo:\n validate\n validate install'
			textoCleanWS = 'Borrar el Worskspace de la tarea jenkins previo a la descarga del código fuente'
			textoCompile = 'Compilar el código fuente generado en la tarea'
			textoDeploy = 'Desplegar el artefacto generado\nEn el caso de no seleccionar ningún servidor, se usará la configuración existente en el profile correspondiente del pom.xml de maven'
			textoSonar = 'Generar las métricas de calidad estática de código a través de SonarQube'
			textoLibrary = 'Generar el inventario de librerías del código fuente descargado'
			textoOverwrite = 'Sobreescribir la configuración por defecto con los valores seleccionados'
			textoBeforeBuild = 'Paso previo a la compilación. Ejecutar línea de comandos (shell)'
			textoAfterBuild = 'Paso posterior a la compilación. Ejecutar línea de comandos (shell)'
			textoBuild = 'Ejecutar línea de comandos (shell)'

			//
			apiUrl = "${JENKINS_URL}job/Administracion/job/configuracion/api/json"
			pathJob = "${JENKINS_HOME}/"
			"${JOB_NAME}".split('/').each {
				pathJob = pathJob + "jobs" + "/" + it + "/"
				jobName = it
			}
			
			jobsFileConfApp = pathJob + "config.yaml"
			
			setConfigurationFile()
			def props = readJSON file: 'configuration.json'
			def choosePortManager= props.property.parameterDefinitions[0].choices[2]
			PortManager = choosePortManager.collect { it }

			def chooseContextPath= props.property.parameterDefinitions[0].choices[3]
			ContextPath = chooseContextPath.collect { it }

			caseJdk = Jenkins.instance.getJDKs()name
			caseMvn = []
			Jenkins.instance.getExtensionList(hudson.tasks.Maven.DescriptorImpl.class)[0].installations.each{
				caseMvn = caseMvn + it.name
			}

			def chooseEnv= props.property.parameterDefinitions[0].choices[0]
			caseEnv = chooseEnv.collect { it }

			def chooseSvrApp= props.property.parameterDefinitions[0].choices[1]
			caseSvrApp = chooseSvrApp.collect { it }

			def chooseJobApp = Jenkins.instance.getAllItems(Job.class).fullName

			File file = new File(jobsFileConfApp)

			if (!file.exists()) {

				properties(
					[parameters([
				
						[$class: 'WReadonlyStringParameterDefinition',
						defaultValue: valorInfo,
						description: textoInfo,
						name: 'info'],
						
						[$class: 'StringParameterDefinition',
						defaultValue: '',
						description: textoUrlGit,
						name: 'urlgit',
						trim: false],
					
						choice(choices: caseMvn,
						description: textoMaven,
						name: 'mvnVersion'),
						
						[$class: 'StringParameterDefinition',
						defaultValue: '', 
						description: textoMvnOption, 
						name: 'mvnOption', 
						trim: false],

						choice(choices: caseJdk,
						description: textoJDK,
						name: 'jdkVersion'),

						[$class: 'ChoiceParameter',
						choiceType: 'PT_SINGLE_SELECT',
						description: textoEnviroments,
						filterLength: 1,
						filterable: false,
						name: 'enviroments',
						randomName: 'choice-parameter-9290271755439',
						script: 
							[$class: 'GroovyScript',
							fallbackScript: [classpath: [],
							sandbox: true,
								script: 'return[""]'],
									script: [classpath: [],
										sandbox: true,
										script: '''
											return ["integracion","preproduccion","produccion"]
										'''
											]
							]
						],
						
						[$class: 'CascadeChoiceParameter',
						choiceType: 'PT_MULTI_SELECT',
						description: textoServer,
						filterLength: 1,
						filterable: false,
						name: 'server',
						randomName: 'choice-parameter-9290273762301',
						referencedParameters: 'enviroments',
						script: [$class: 'GroovyScript',
						fallbackScript: [classpath: [],
						sandbox: false,
						script: 'return[""]'],
						script: [classpath: [],
						sandbox: true,
						script: '''
							import groovy.json.JsonSlurper
							
							def jobName = ''
							def build = Thread.currentThread().toString()
							def regexp= ".+?job/([^/]+)"
							def match = build  =~ regexp
							match.each{
								jobName = jobName + it[1].replace("%20"," ") + "/"
							}
							
							pathJob = System.getenv('HOME') + "/"
							configurationFile = pathJob + 'workspace/' + jobName
							
							def inputFile = new File(configurationFile + 'configuration.json')
							def inputJSON = new JsonSlurper().parse(inputFile)

							def chooseInt= inputJSON.property.parameterDefinitions[0].choices[4]
							def caseInt = chooseInt.collect { it }

							def choosePre= inputJSON.property.parameterDefinitions[0].choices[5]
							def casePre = choosePre.collect { it }

							def choosePro= inputJSON.property.parameterDefinitions[0].choices[6]
							def casePro = choosePro.collect { it }

							if (enviroments.equals("integracion")){
								return caseInt
							}else if (enviroments.equals("preproduccion")) {
								return casePre
							}else if (enviroments.equals("produccion")) {
								return casePro
							}

						''']]
						],
						
						choice(choices: chooseSvrApp,
						description: textoSvrApp,
						name: 'svrApp'),
						
						choice(choices: PortManager.join('\n'),
						description: textoPuerto,
						name: 'portManager'),
						
						choice(choices: ContextPath.join('\n'),
						description: textoContexto,
						name: 'contextPath'),
						
						[$class: 'StringParameterDefinition',
						defaultValue: '',
						description: textoBeforeBuild,
						name: 'beforeBuildTask',
						trim: false],
						
						[$class: 'StringParameterDefinition',
						defaultValue: '',
						description: textoAfterBuild,
						name: 'afterBuildTask',
						trim: false]
					
					])]
				)
				
				setFicheroYaml()
				println "Fichero de configuración creado"
				build job: "${JOB_NAME}", parameters: [
					[$class: 'BooleanParameterValue', name: 'cleanWs', value: 'false'],
					[$class: 'BooleanParameterValue', name: 'compile', value: 'false'],
					[$class: 'BooleanParameterValue', name: 'deploy', value: 'false'],
					[$class: 'BooleanParameterValue', name: 'sonar', value: 'false'],
					[$class: 'BooleanParameterValue', name: 'library', value: 'false'],
					[$class: 'StringParameterValue', name: 'urlgit', value: urlgit]
				]
				currentBuild.result = 'ABORTED'
				
			}else {
			
				dataJobConf = readYaml file: jobsFileConfApp
				valorEnviroment = []
				valorEnviroment = valorEnviroment + dataJobConf.enviroment
				valorEnviroment = valorEnviroment + (dataJobConf.enviroment == "preproduccion"?"integracion":"preproduccion")
				valorPortManager = (dataJobConf.portManager == null?PortManager.join('\n'):dataJobConf.portManager)
				valorContextPath = (dataJobConf.contextPath == null?ContextPath.join('\n'):dataJobConf.contextPath)
				valorBeforeBuildTask = (dataJobConf.beforeBuildTask == null?'':dataJobConf.beforeBuildTask)
				valorAfterBuildTask = (dataJobConf.afterBuildTask == null?'':dataJobConf.afterBuildTask)
				
				def usuarioActual = currentBuild.getBuildCauses('hudson.model.Cause$UserIdCause')
				if(usuarioActual.userId[0] != null){
					def instance = Jenkins.getInstance()
					writePermission = instance.getAuthorizationStrategy().hasPermission(usuarioActual.userId[0], Jenkins.ADMINISTER).toString()
				}else{
					writePermission = false
				}
				
				if (writePermission) {
					claseUrlGit = 'StringParameterDefinition'
				}else{
					claseUrlGit = 'WReadonlyStringParameterDefinition'
				}
				
				properties(
					[parameters([
						
						[$class: 'WReadonlyStringParameterDefinition',
						defaultValue: valorInfo,
						description: textoInfo,
						name: 'info'],
						
						[$class: claseUrlGit,
						defaultValue: dataJobConf.urlgit,
						description: textoUrlGit,
						name: 'urlgit',
						trim: false],
						
						listGitBranches(
						branchFilter: 'refs/heads/(release.*)',
						defaultValue: 'master',
						name: 'branch',
						type: 'BRANCH',
						selectedValue: 'TOP',
						remoteURL: dataJobConf.urlgit,
						credentialsId: 'gitlab-token'),
						
						booleanParam(defaultValue: true, 
						description: textoCleanWS,
						name: 'cleanWs'),
						
						booleanParam(defaultValue: true, 
						description: textoCompile,
						name: 'compile'),
						
						booleanParam(defaultValue: false,
						description: textoDeploy,
						name: 'deploy'),
						
						booleanParam(defaultValue: false,
						description: textoSonar,
						name: 'sonar'),
						
						booleanParam(defaultValue: false,
						description: textoLibrary,
						name: 'library'),
						
						choice(choices: dataJobConf.mvnversion,
						description: textoMaven,
						name: 'mvnVersion'),
						
						string(defaultValue: dataJobConf.mvnOption, 
						description: textoMvnOption, 
						name: 'mvnOption', 
						trim: false),
						
						choice(choices: dataJobConf.jdkversion,
						description: textoJDK,
						name: 'jdkVersion'),
						
						choice(choices: valorEnviroment,
						description: textoEnviroments,
						name: 'enviroments'),
						
						extendedChoice(defaultValue: dataJobConf.server,
						description: textoServer, 
						name: 'server', 
						type: 'PT_MULTI_SELECT', 
						visibleItemCount: 10,
						groovyScript: '''
							import org.yaml.snakeyaml.Yaml
							import groovy.json.JsonSlurper
							
							def jobName = ''
							def build = Thread.currentThread().toString()
							def regexp= ".+?job/([^/]+)"
							def match = build  =~ regexp
							match.each{
								jobName = jobName + it[1].replace("%20"," ") + "/"
							}
							
							Yaml parser = new Yaml()
							pathJob = System.getenv('HOME') + "/"
							configurationFile = pathJob + 'workspace/' + jobName
							jobName.split('/').each {
								pathJob = pathJob + "jobs" + "/" + it + "/"
								jobName = it
							}
							jobsFileConfApp = pathJob + "config.yaml"
							
							def dataJobConf = parser.load((jobsFileConfApp as File).text)
							def enviroment = dataJobConf.enviroment
							
							def inputFile = new File(configurationFile + 'configuration.json')
							def inputJSON = new JsonSlurper().parse(inputFile)

							def chooseInt= inputJSON.property.parameterDefinitions[0].choices[4]
							def caseInt = chooseInt.collect { it }

							def choosePre= inputJSON.property.parameterDefinitions[0].choices[5]
							def casePre = choosePre.collect { it }

							def choosePro= inputJSON.property.parameterDefinitions[0].choices[6]
							def casePro = choosePro.collect { it }

							if (enviroment.equals("integracion")){
								return caseInt
							}else if (enviroment.equals("preproduccion")) {
								return casePre
							}else if (enviroment.equals("produccion")) {
								return casePro
							}
						'''
						),
						
						choice(choices: dataJobConf.svrapp,
						description: textoSvrApp,
						name: 'svrApp'),
						
						choice(choices: valorPortManager,
						description: textoPuerto,
						name: 'portManager'),
						
						choice(choices: valorContextPath,
						description: textoContexto,
						name: 'contextPath'),
						
						[$class: 'StringParameterDefinition',
						defaultValue: valorBeforeBuildTask,
						description: textoBeforeBuild,
						name: 'beforeBuildTask',
						trim: false],
						
						[$class: 'StringParameterDefinition',
						defaultValue: valorAfterBuildTask,
						description: textoAfterBuild,
						name: 'afterBuildTask',
						trim: false],
						
						booleanParam(defaultValue: false,
						description: textoOverwrite,
						name: 'overwrite')
						
					])]
				)
			} 

//Borrado de workspace
	stage ('Clean WORKSPACE') {

		script {
			if (params.cleanWs) { 
				cleanWs(patterns: [[pattern: '', type: 'INCLUDE']])
				sh label: '', script: "rm -r '${WORKSPACE}'/*"
			}
		}
		
	}

//Descaga del codigo del repositorio
	stage('Download') {
        
		git branch: "${params.branch}", url: "${params.urlgit}", credentialsId: 'gitlab-token'
		setConfigurationFile()

	}

//Preparacion de las instrucciones maven a ejecutar segun la eleccion de parametros
	stage('Configuration'){
	
		script {
			if(params.compile || params.deploy || params.library){
				//Creacion de la instruccion maven a utilizar
				mvnHome = tool "${mvnVersion}"
				javaHome = tool "${jdkVersion}"
				try{
					sh label: '', script: "export JAVA_HOME=${javaHome} && ${mvnHome}/bin/mvn -version"
				}catch(Exception exp){
					javaHome = "/opt/jdk1.7.0_55"
				}
				javaSite = javaHome
				if(javaSite == "/opt/jdk1.5.0_19" || javaSite == "/opt/jdk1.6.0_20" || javaSite == "/opt/jdk1.6.0_29"){
					javaSite = "/opt/jdk1.7.0_55"
				}
				instruccionMaven = "export JAVA_HOME=${javaHome} && ${mvnHome}/bin/mvn clean -Dmaven.test.skip=true package"
				instruccionMavenSite = "export JAVA_HOME=${javaSite} && ${mvnHome}/bin/mvn clean -Dmaven.test.skip=true package"
				if(library.equals("true")){
					instruccionMavenSite = instruccionMavenSite + " ${mvnOption} site"
				}
				instruccionMaven = instruccionMaven + " ${mvnOption}"
				if(deploy.equals("true") && "${params.server}" == ""){
					if (svrApp == "tomcat" || svrApp == "tomcat6") {
						instruccionMaven = instruccionMaven + " ${svrApp}:list ${svrApp}:redeploy ${svrApp}:list"
					}
					else if (svrApp == "tomcat7" || svrApp == "tomcat8" || svrApp == "tomcat9") {
						instruccionMaven = instruccionMaven + " tomcat7:redeploy"
					}
					else if (svrApp == "wildfly") {
						instruccionMaven = instruccionMaven + " ${svrApp}:deploy"
					}
				}
				instruccionMaven = instruccionMaven + " -P${enviroments}"
				instruccionMavenSite = instruccionMavenSite + " -P${enviroments}"

				setFilterPom('pom.xml')
				deleteContext('pom.xml')

				def pomXML = readMavenPom file: 'pom.xml'
				def pomXMLModulos =  pomXML.modules
				
				if(deploy.equals("true")){
					//Buscamos el context.xml para ver el path que tiene puesto y si es el mismo que el seleccionado
					sh label: '', script: "find . -type f -name 'context.xml' > ficheroContext.txt"
					def rutaFicheroContext = readFile ("./ficheroContext.txt")
					def lineas = rutaFicheroContext.readLines()
					for (line in lineas) {
						line = line.substring(1, line.length())
						def xmlFile = "${WORKSPACE}" + line
						def xml = new XmlParser().parse(xmlFile)
						if(xml['@path'] != null && xml['@path'] != '${tomcat.contexto}'){
							xml['@path'] = '${tomcat.contexto}'
							new XmlNodePrinter(new PrintWriter(new FileWriter(xmlFile))).print(xml)
						}
					}
					
					//Nombre de los war a desplegar
					warDesplegarList = []
					nombreDespliegue = pomXML.build.finalName
					if(nombreDespliegue == '${project.artifactId}'){
						nombreDespliegue = pomXML.artifactId
					}
					if(nombreDespliegue != null){
						warDesplegarList.add(nombreDespliegue)
					}
					//echo nombreDespliegue
					
					pomXMLModulos.each {
						pomXMLModulo = readMavenPom file: it + "/pom.xml"
						nombreDespliegueModuloPackaging = pomXMLModulo.packaging
						if(nombreDespliegueModuloPackaging == 'war'){
							nombreDespliegueModulo = pomXMLModulo.build.finalName
							if(nombreDespliegueModulo.contains('$')){
								nombreDespliegueModuloTMP = nombreDespliegueModulo.replace('$', '').replace('{', '').replace('}', '').replace('project.', '')
								try{
									nombreDespliegueModulo = pomXMLModulo."${nombreDespliegueModuloTMP}"
								}catch(Exception exp){
									try{
										nombreDespliegueModulo = pomXML."${nombreDespliegueModuloTMP}"
									}catch(Exception exp1){
										nombreDespliegueModulo = readMavenPom().getProperties().getProperty("${nombreDespliegueModuloTMP}")
									}
								}
							}
							//echo nombreDespliegueModulo
							if(nombreDespliegueModulo != null && !warDesplegarList.contains(nombreDespliegueModulo)){
								warDesplegarList.add(nombreDespliegueModulo)
							}
						}
					}
				}
				if(library.equals("true")){
					setPluginsPom('pom.xml')
					pomXMLModulos.each {
						setPluginsPom(it + '/pom.xml')
					}
				}
			}
		}
		
	}

//Ejecucion de tareas antes
	stage('Task Before Build') {
	
		script {
			if (beforeBuildTask.trim() != '' && (params.compile || params.deploy)) {
				sh script: beforeBuildTask.trim()
			}
		}
		
	}

//Compilacion
	stage('Compile') {

		script {
			
			if (params.compile && !params.deploy) {
				sh label: '', script: "${instruccionMaven}"
			}
		}
		
	}

//Despliegue de compilado
	stage('Deploy') {

		script {
			if (params.deploy) {
				sh label: '', script: "${instruccionMaven}"
			
				if ("${params.server}" != "") {
					def paramServer = "${params.server}".tokenize(",")
					def contextRuta = "${params.contextPath}"

					idFile = "/opt/jenkins/credenciales_servidores/${enviroments}/id.txt"
					credentials = readYaml file: idFile

					for (server in paramServer)  {

						if(svrApp == "wildfly"){
							sh label: '', script: "export JAVA_HOME=${javaHome} && ${mvnHome}/bin/mvn -Dmaven.test.skip=true -Dwildfly.nombreServidor=" + server + " -Dwildfly.puertoServidor=${portManager} ${svrApp}:deploy -P${enviroments}"
						}
						else {
							if( svrApp == "tomcat") {
								svrApp = "tomcat5"
							}
							warDesplegarList.each {
								warDesplegar = it
								deploy adapters: ["${svrApp}"(credentialsId: credentials,
								path: '', url: "http://" + server + ":${portManager}"),],
								contextPath: "${contextRuta}" + "${warDesplegar}", war: "**/${warDesplegar}.war"
							}
						}
					}
				}
			}
		}

	}

//Ejecucion del sonar scanner
	stage('SonarQube') {

		script {
			if (params.sonar) {
				def scannerHome = tool 'SonarQubeScanner'
				sh label: '', script: "/opt/jdk1.8.0_77/bin/java -jar /opt/jenkinsOC_profiles/SonarProperties.jar ${JOB_BASE_NAME}"
				withSonarQubeEnv('sonarQube_cmaot') {
					sh label: '', script: "export JAVA_HOME=/opt/jdk1.8.0_77/ && ${scannerHome}/bin/sonar-scanner '-Dproject.settings=${WORKSPACE}/sonar-project.properties'"
				}
			}
		}

	}

//Persistencia de librerias usadas en el compilado
	stage('Library Persistence') {
	
		script {
			if (params.library) {
				if (compile.equals("false") && deploy.equals("false")) {
					sh label: '', script: "${instruccionMavenSite}"
				}else{
					sh label: '', script: "export JAVA_HOME=${javaSite} && ${mvnHome}/bin/mvn site"
				}
				sh label: '', script: "echo ${JOB_BASE_NAME} > '${WORKSPACE}/job.txt'"
				sh label: '', script: "/opt/jdk1.8.0_77/bin/java -jar /opt/jenkinsOC_profiles/Dependencias.jar '${WORKSPACE}/job.txt'"
			}
		}

	}

//Ejecucion de tareas despues
	stage('Task After Build') {

		script {
			if (afterBuildTask.trim() != '' && (params.compile || params.deploy)) {
				sh script: afterBuildTask.trim()
			}
		}

	}

//Sobreescribir la configuracion por defecto
	stage("Write Configuration") {
	
		script {
			if (params.overwrite) {
				setFicheroYaml()
				println "Fichero de configuración modificado"
			}
		}
	}

//Autorefresco de la propia tarea para coger los cambios de configuracion
	stage ("Refresh Configuration") {
	
		script {
			if (params.overwrite) {
				build job: "${JOB_NAME}", parameters: [
					[$class: 'BooleanParameterValue', name: 'cleanWs', value: 'false'],
					[$class: 'BooleanParameterValue', name: 'compile', value: 'false'],
					[$class: 'BooleanParameterValue', name: 'deploy', value: 'false']
				]
			}
		}
	
	}

     }
  }
}
