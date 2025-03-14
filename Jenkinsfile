node {
	checkout scm
	def PIPENV = "~/.local/bin/pipenv"

	stage("Build") {
		dir("restapi") {
			sh "pip3 install pipenv"
			sh "${PIPENV} install"
		}
	}

	stage("Test") {
		def PORT = 3306
		def ROOT = "secret_root"
		def DB	 = "movieapp"
		def USER = "movieapp"
		def PASS = "secret_user"
		try {
			docker.image("mariadb:latest").withRun("-p ${PORT}:${PORT} -e MYSQL_ROOT_PASSWORD=${ROOT} -e MYSQL_DATABASE=${DB} -e MYSQL_USER=${USER} -e MYSQL_PASSWORD=${PASS}") { c ->
				docker.image("mariadb:latest").inside("--link ${c.id}:db") {
            		sh 'while ! mysqladmin ping -hdb --silent; do sleep 1; done'
        		}
				dir("restapi") {
					withEnv(["DATABASE=${DB}","DBUSER=${USER}","PASSWORD=${PASS}"]){
						sh "${PIPENV} run python tests/MoviesTest.py"
						sh "${PIPENV} run python tests/CategoriesTest.py"
					}
				}
			}
		}
		catch (Exception e) {
			error "Unable to launch tests.Probably because something is running on port 3306."
		}
	}

	stage("Deploy") {
		docker.withRegistry("" , "DockerCreds") {
			dir("restapi") {
				docker.build("${params.registry}/restapi").push("latest")
			}
			dir("webapp") {
				docker.build("${params.registry}/webapp").push("latest")
			}
		}
	}

	stage("Run") {
		try {
			sshagent(["DevServer"]) {
				def remote = "${params.remoteUser}@${params.remoteHost}"
				def option = "-o StrictHostKeyChecking=no"
				sh "scp ${option} docker-compose.yaml .env ${remote}:~"
				sh "ssh ${option} ${remote} docker-compose pull"
				sh "ssh ${option} ${remote} docker-compose up -d"
			}
		}
		catch (Exception e) {
			error "Unable to start services.Probably because something is running on port 80 or 8000."
		}
		finally {
			sh "docker system prune -f"
		}
	}

}
