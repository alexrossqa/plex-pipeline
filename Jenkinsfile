pipeline {
    agent any

    triggers {
        cron('0 2 * * *')
    }

    environment {
        LAST_RUN      = '1770000000'
        LAST_RUN_FILE = 'C:\\Jenkins\\plex-last-run.txt'
        LOG_DIR       = 'C:\\Jenkins\\Logs\\PlexSync'
        GDRIVE_FOLDER = 'G:\\Other computers\\MINI\\#FILMS\\'
        HDD_FOLDER    = 'E:\\#FILMS\\'
        PLEX_TOKEN    = credentials('plex-token')
        TMDB_API_KEY  = credentials('tmdb-api-key')
		PLEX_URL      = 'http://127.0.0.1:32400'
		GMAIL_USERNAME    = credentials('gmail-username')
		GMAIL_APP_PASSWORD = credentials('gmail-app-password')
		MAIL_RECIPIENTS = 'alex.ross.qa@gmail.com,78matt.grant@gmail.com,contact@jasminesoliman.com'
    }

    stages {

        stage('Checkout') {
            steps {
                dir('plex-sync-scripts') {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/master']],
                        userRemoteConfigs: [[
                            url: 'https://github.com/alexrossqa/plex-sync-scripts.git'
                        ]]
                    ])
                }
                dir('plex-rest-assured') {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: '*/main']],
						extensions: [[$class: 'CleanBeforeCheckout']],
                        userRemoteConfigs: [[
                            url: 'https://github.com/alexrossqa/plex-rest-assured.git'
                        ]]
                    ])
                }
            }
        }

        stage('Upload to GDrive') {
            steps {
                powershell '''
                    & "$env:WORKSPACE\\plex-sync-scripts\\uploadToGDrive.ps1" `
                        -SourceFolder $env:HDD_FOLDER `
                        -DestFolder   $env:GDRIVE_FOLDER `
                        -LogDir       $env:LOG_DIR
                '''
            }
        }

        stage('Wait for GDrive to Stabilise') {
            steps {
                powershell '''
                    & "$env:WORKSPACE\\plex-sync-scripts\\Sync-AndWait.ps1" `
                        -WatchFolder            $env:GDRIVE_FOLDER `
                        -StabilityWindowSeconds 60 `
                        -PollIntervalSeconds    15 `
                        -TimeoutMinutes         180 `
                        -LogDir                 $env:LOG_DIR
                '''
            }
        }

        stage('Download to HDD') {
            steps {
                powershell '''
                    & "$env:WORKSPACE\\plex-sync-scripts\\downloadToHdd.ps1" `
                        -SourceFolder $env:GDRIVE_FOLDER `
                        -DestFolder   $env:HDD_FOLDER `
                        -LogDir       $env:LOG_DIR
                '''
            }
        }

        stage('Wait for HDD to Stabilise') {
            steps {
                powershell '''
                    & "$env:WORKSPACE\\plex-sync-scripts\\Sync-AndWait.ps1" `
                        -WatchFolder            $env:HDD_FOLDER `
                        -StabilityWindowSeconds 60 `
                        -PollIntervalSeconds    15 `
                        -TimeoutMinutes         180 `
                        -LogDir                 $env:LOG_DIR
                '''
            }
        }

        stage('Run Plex API Tests') {
            steps {
                bat 'mvn clean test -Dplex.baseUrl=%PLEX_URL% -Denv=ad -Dgroups=daily -DlastRun=%LAST_RUN% -Dplex.token=%PLEX_TOKEN% -Dtmdb.baseUrl=https://api.themoviedb.org/3 -Dtmdb.apiKey=%TMDB_API_KEY% -Dgmail.username=%GMAIL_USERNAME% -Dgmail.password=%GMAIL_APP_PASSWORD% -Dmail.recipients=%MAIL_RECIPIENTS% -f %WORKSPACE%\\plex-rest-assured\\pom.xml' 
            }
            post {
                success {
                    powershell '''
                        $ts = [DateTimeOffset]::UtcNow.ToUnixTimeSeconds()
                        Set-Content -Path $env:LAST_RUN_FILE -Value $ts -Encoding UTF8
                        Write-Output "Timestamp written: $ts"
                    '''
                }
            }
        }

    }
}