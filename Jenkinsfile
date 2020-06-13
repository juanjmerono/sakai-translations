properties([[$class: 'BuildDiscarderProperty',
                strategy: [$class: 'LogRotator', numToKeepStr: '5']],
                pipelineTriggers([cron('@midnight')]),
                ])

node {

	// Clean the workspace
	stage ('Cleanup') {
		step([$class: 'WsCleanup'])
	}

	// First checkout the code
	stage ('Checkout') {
	
		// Checkout the source from sakai-translations.
		checkout scm
	   	// Checkout code from sakai repository
	   	dir('sakai') {
	   		git ( [url: 'https://github.com/sakaiproject/sakai.git', branch: env.BRANCH_NAME] )
	   	}
	   	// Move files inside sakai/l10n folder to work properly
	   	dir('.') {
	   		sh 'rm -rf sakai/l10n'
	   		sh 'mkdir sakai/l10n'
	   		sh 'cp l10n/* sakai/l10n/.'
	   	}
	   	
	}

	// Read properties after checkout
	def props = readProperties  file: 'translation.properties'
	def transifex_project = props['PROJECTNAME']
	def locales = props['LOCALES'].split(',')

   	// Now run init transifex 
   	stage ('Init Transifex') {
   		env.TRANSIFEX_SAKAI_PROJECTNAME=transifex_project
	   	dir ('sakai/l10n') {
	   		sh 'echo -e "\n" | python tmx.py init'
	   	}
	}
	
	// Now upload translations to transifex
	stage ('Upload Translations') {
		env.TRANSIFEX_SAKAI_PROJECTNAME=transifex_project
	   	dir ('sakai/l10n') {
			sh "python tmx.py update --java2po"
   			sh "python tmx.py upload -m -v"
	   	}
	}   	

	// Now download translations from transifex
	stage ('Download Translations') {
		env.TRANSIFEX_SAKAI_PROJECTNAME=transifex_project
	   	dir ('sakai') {
   			for (int i=0; i<locales.size(); i++) {
   				sh "cd l10n;python tmx.py download -r -u -c -l ${locales[i]};cd .."
	   			sh "git add '*_${locales[i]}.properties'"
	   			sh "git diff --staged --ignore-space-at-eol -- '*_${locales[i]}.properties' > ../translation_${locales[i]}.patch"
				sh "git reset HEAD"
   			}
	   	}
	}   	
	   	
	stage ('Publish Patches') {
		for (int i=0; i<locales.size(); i++) {
			publishHTML(
				[allowMissing: false, 
				 alwaysLinkToLastBuild: false, 
				 keepAll: false, 
				 reportDir: '.', 
				 reportFiles: 'translation_' + locales[i] + '.patch', 
				 reportName: 'TranslationPatch_' + locales[i] ])
		}
	}
}
