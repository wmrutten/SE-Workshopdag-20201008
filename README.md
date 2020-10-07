# Furore SE Workshopdag CI/CD

## Stap 1 - Forken
 -  Fork deze **GitHub repository**
  
## Stap 2 - Nieuw Azure project aanmaken 
 - Ga naar https://dev.azure.com/ en maak een nieuwe **Azure DevOps Organization** aan
	 - Vul een naam in voor jouw Azure DevOps Organization
	 - Locatie mag je zelf kiezen
 - Creëer vervolgens een **Private project** binnen de zojuist gemaakte organisatie

Je hebt nu een leeg Azure DevOps project. We gaan hierbinnen een **build pipeline** en een **release pipeline** maken. In de build pipeline checken we het project uit van GitHub en gaan deze in Azure builden. Deze build gaan we vervolgens in de release pipeline releasen op een Azure App Service om deze vervolgens te deployen.

## Stap 3 - Nieuwe build pipeline aanmaken
We gaan een build pipeline aanmaken en deze koppelen aan jouw fork van dit project. Op deze manier wordt jouw GitHub repository de codebase voor deze pipeline.

- Ga binnen jouw net aangemaakte project naar het tablad **Pipelines** en kies dan **Pipelines**
- Creëer een niewe pipeline
![create_build_pipeline](/.attachments/create_build_pipeline.jpg)
- Kies onderaan voor **Use the classic editor**
![classic_editor](/.attachments/classic_editor.jpg)
- Kies als source **GitHub**
- Kies **Authorize using OAuth** en doorloop de stappen op het pop-up scherm om AzurePipelines te autoriseren
- Selecteer vervolgens de geforkte repository
- Als laatste wordt gevraagd of je een template wilt selecteren om jouw build te definieren. In deze templates zijn al wat taken (build, publish, enzovoort) voor jou ingevuld. Aangezien dit een workshopdag is, gaan wij deze zelf definieren. Kies daarom bovin voor **Empty job**
![empty_job](/.attachments/empty_job.jpg)

## Stap 4 - Eerste build maken
In de zojuist gemaakte build pipeline zie je een **Agent job 1**. Deze agent voert **Get sources** uit om de code van GitHub af te halen. In deze stap gaan we **Tasks** aan de agent toevoegen zodat deze de dependencies in de code **restored** en de code vervolgens **build**.
![add_task](/.attachments/add_task.jpg)
- Voeg twee **.NET Core** tasks toe aan de agent. Vul bij beide tasks bij **Path to project**, `**/*.csproj` in
	- Een task om dependencies te restoren
	- Een task om de code te builden
- Klik op **Save & queue** om de build pipeline op te slaan
- Klik op vervolgens op **Save & run**. Azure gaat nu een build maken van jouw code (duurt ongeveer 5 minunten). Als je op de agent klikt, kan je zien waar hij op dat moment mee bezig is.
![agent_run](/.attachments/agent_run.jpg)


## Stap 5 - Publish Artifact naar de Pipeline
Nu we een build kunnen maken, moet we twee tasks aan de agent toevoegen om de build te **publishen** en als **artifact** klaarzetten voor deployment. Bij het publishen wordt de build en zijn dependencies in een folder klaargezet als artifact. Dit artifact dient weer beschikbaar gesteld te worden aan de pipeline, zodat deze straks verderop in de pipeline opgepakt kan worden.
- Voeg, net als in de vorige stap, een **.NET Core** task toe aan de agent om het project te **publishen**
	- Bij **Arguments** vul `-o "$(build.artifactstagingdirectory)"` in. Dit is de folder waarin jouw build en dependencies terecht komen, ofwel jouw **Artifact**
	- Laat de rest op default

![publish](/.attachments/publish.jpg)

- Voeg vervolgens een **Publish build artifacts** toe
	- Bij **Path to publish** vul `$(build.artifactstagingdirectory)` in. Dit is de folder waarin jouw artifact staat die je aan de pipeline beschikbaar wilt stellen
	- Kies een mooie naam voor jouw artifact
	- Laat de rest op default
![publish_artifact](/.attachments/publish_artifact.jpg)

- Klik op **Save & queue** om de build pipeline op te slaan
- Klik op vervolgens op **Save & run**. Azure gaat nu een build maken van jouw code publishen als artifact .  Als je op de agent klikt, kan je zien waar hij op dat moment mee bezig is. Je kan alvast door met de volgende stap

## Stap 6 - Eenmalig App services instellen 
Nu is het tijd om een **App service** te maken en in te stellen. Dit is de plek (server) waar de app straks komt te draaien.
- Ga naar https://portal.azure.com/ en maak een nieuwe **App Services** aan
![app_services](/.attachments/app_services.jpg)

- Selecteer jouw subscription
- Create new Resource Group (geef leuke naam)
- Geef een naam aan jouw instance (dit wordt de URL)
- Zie voor de rest screenshot
![app_service_settings](/.attachments/app_service_settings.jpg)

- **BELANGRIJK!** Klik onderaan op **Change size** om een **Free tier** te kiezen onder **Dev/Test**
![free_tier](/.attachments/free_tier.jpg)
- Klik vervolgens op **Review + create** en daarna op **Create** on de App service aan te maken
- Je krijgt een melding zodra de App service beschikbaar is
![app_service_available](/.attachments/app_service_available.jpg)


## Stap 7 -Release pipeline aanmaken
Nu we een App service beschikbaar hebben waarop we ons artifact kunnen gaan hosten, kunnen we de release pipeline gaan maken. Binnen de release pipeline geven we aan welk artifact we willen releasen en waar we deze willen deployen.
- Ga binnen jouw project (azure.dev) naar het tablad **Pipelines** en kies dan **Releases**
- Maak een **New pipeline**
- Selecteer **Empty job**
![empty_job_release](/.attachments/empty_job_release.jpg)
- Hernoem stage name naar **Develop**
- Klik op **Add an artifact** om jouw artifact toe te voegen aan de release pipeline
- Klik vervolgens op **1 job, 0 task** onder Develop
- Voeg onder de **Agent job** een **Azure App Service deploy ** task toe. Hiermee kunnen we het artifact deployen op onze eerder gemaakte App service
- Kies de juiste subscription en App service name
- Laat **Package or folder** op de default waarde staan
- Druk op **Save** om de release pipeline op te slaan
![deploy_app_task](/.attachments/deploy_app_task.jpg)

## Stap 7 - Release (deploy naar App services)
Nu we eindelijk de build en release pipeline hebben gebouwd, kunnen we onze artifact gaan releasen.
- Ga naar het tabblad **Pipelines** en en kies dan **Releases**
- Klik op **Create release**
- Klik op **Create**
- Er wordt gelijk een release gestart
![release_running](/.attachments/release_running.jpg)
- Ga naar de App service in https://portal.azure.com/, daar zie je de URL naar jou app als de release klaar is
	- https://APPSERVICENAME.azurewebsites.net
![url](/.attachments/url.jpg)

## Stap 8 - Voeg unit tests toe
We hebben zojuist een app gereleased zonder dat we deze 'getest' hebben. Gelukkig bevat het project unit tests. Het uivoeren van deze unit tests gaan we toevoegen aan onze build pipeline. De tests gaan we uitvoeren na de build task en voor de publish task.
- Ga naar het tabblad **Pipelines** en kies dan **Pipelines**
- Klik op jouw build pipeline
- Klik vervolgens op **Edit** om jouw build pipeline aan te passen
![edit_pipeline](/.attachments/edit_pipeline.jpg)
- Voeg onder de Agent job een nieuwe **.NET Core** task toe om de Unit tests van het project te runnen (tussen de build en de publish)
	- Bij **Path to project(s)** vul `**/*UnitTests/*.csproj` in. Dit is de folder waarin de unit tests staan
- Sleep de task naar de juiste plek
![add_unit_test](/.attachments/add_unit_test.jpg)

- Ga tot slot weer terug naar het tabblad **Pipelines** en kies dan **Pipelines**
- Klik jouw build pipeline aan en klik **Run pipeline**. De build pipeline gaat starten en zal vast lopen op de test. Een software engineer (uiteraard niet van Furore) heeft namelijk een fout in de code gemaakt... In de volgende stap gaan we dit fixen

## Stap 9 - GitHub commit trigger
Een goede software engineer is lui en wil daarom graag zo veel mogelijk automatiseren. Dat is precies wat we in deze stap gaan doen. Het handmatig aanzetten van de build en release pipelines is zonde tijd. In deze stap gaan we triggers instellen waardoor de build pipeline en daarna de release pipeline worden gestart, zodra jij een nieuwe commit naar GitHub pusht.
- Build trigger (Continuous Integration)
	- Ga naar het tabblad **Pipelines** en kies dan **Pipelines**
	- Klik op jouw build pipeline
	- Klik vervolgens op **Edit** om jouw build pipeline aan te passen
	- Naast het tabblad **Tasks** zie je het tabblad **Triggers**, klik hierop
	- Vink vervolgens **Enable continious integration** aan en druk kies **Save** onder het kopje **Save & queue**
![ci](/.attachments/ci.jpg)
- Release trigger (Continious Delivery)
	- Ga naar het tabblad **Pipelines** en kies dan **Releases**
	- Klik vervolgens op **Edit** om jouw release pipeline aan te passen
	- Rechtsboven jouw artifact zie je een trigger logo, klik hierop
![cd_trigger](/.attachments/cd_trigger.jpg)
	- Zet vervolgens **Create a release every time a new build is availabele** op enabled, druk op het kruisje en **Save** vervolgens de release pipeline
![cd](/.attachments/cd.jpg)

## Stap 10 - Fix de fout en commit
Nu is het tijd om de fout te fixen en de oplossing naar GitHub te pushen. Deze push zal een nieuwe build en release triggeren.
- Fix de fout
- Push de oplossing naar GitHub
- Ga terug naar Azure DevOps en zie dat er een nieuwe build en release gestart zijn

## Tijd over?
Probeer de pipelines eens te bouwen middels YAML files, zie https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema

## Valsspelen?
We hebben nu handmatig een CI/CD pipeline opgezet die getriggerd wordt door een push naar onze GitHub repo. Microsoft zou Microsoft niet zijn als je bovenstaande pipeline ook automatisch kan laten generen, zie [DevOps Starter](https://docs.microsoft.com/en-us/azure/devops-project/azure-devops-project-github).
