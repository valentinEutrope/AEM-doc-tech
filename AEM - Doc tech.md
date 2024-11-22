
# Doc techniques

## Build

#### Plugin
Installez le plugin"VSCode AEM Sync" sur VSCode, ce dernier permettra de synchroniser le code entre l'IDE et CRX. Entrez dans la config (@ext:yamato-ltd.vscode-aem-sync) et cochez le "Autopush" pour que le code s'importe dans AEM à chaque sauvegarde de fichier. 

#### MVN
Les modifications java doivent se build pour apparaître dans AEM à l'instar du front qui peut s'exporter via sync. Pour build le java, il faut build le dossier core à l'aide de maven. Lancer la commande : `cd core && mvn clean install -PautoInstallBundle` 

> A noter : vous pouvez build tout le projet (core, content, frontend, tests et apps) en lançant ```mvn clean install -PautoInstallSinglePackage``` directement depuis la racine du projet

## Composants

Section traitant de la création d'un composant, de son architecture et de la manière d'y injecter du js et du style. On se base ici sur le site d'exemple WKND. 

### Créer un nouveau composant

 1. Créer un dossier `mycomponent` dans `ui.apps/src/main/content/jcr_root/apps/wknd/components/content`
 2. Créer un fichier `mycomponent.html` vous pouvez y mettre le code simple suivant : 
 ```html
 <div class="mycomponent">
	 Hello
 </div>
 ```
 3. Créer un fichier `.content.xml` qui contiendra toute les infos nécessaire pour l'éditeur AEM. Il faut définir à minima son **primaryType** en `cq:Component`, son **title**, le **group** auquel il appartient et un composant du core sur lequel il se base (on choisit en général un composant simple comme "container"). Vous pouvez y mettre le code exemple suivant : 
 ```xml
 <?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0" xmlns:cq="http://www.day.com/jcr/cq/1.0"  xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
jcr:primaryType="cq:Component"
jcr:title="MyComponent"
sling:resourceSuperType="core/wcm/components/container/v1/container"
componentGroup="WKND.Content"/>
 ```  
 > Notez que dans l'exemple, nous utilisons le groupe WKND, adaptez le à votre environnement au besoin.
 > 
 > A la place d'utiliser la propriété resourceSuperType pour se baser sur un composant et hériter de sa boîte de dialogue, vous pouvez créer un dossier `_cq_dialog` dans lequel vous pouvez décrire votre propre xml. Si vous ne souhaitez pas avoir de dialog, vous pouvez simplement créer le dossier vide pour que votre composant s'affiche dans la liste. Sans ce dossier ou sans un resourceType, votre composant ne s'affichera pas.
 
 4. Modifier le fichier xml global du groupe. Ce dernier liste tout les composants de votre groupe. Il faut y ajouter le votre. Par exemple, dans notre groupe WKND, nous allons nous rendre sur le fichier `ui.apps/src/main/content/jcr_root/apps/wknd/clientlibs/clientlib-base/.content.xml` et ajouter dans la liste `embed` notre composant : `wknd.mycomponent`. 
 
 > écrire le nom du dossier et non le title du composant
 
 6. Synchronisez votre code ("export to AEM server") et rendez-vous sur l'éditeur AEM. Sélectionner votre site ainsi qu'une page à éditer pour avoir la liste des composants à gauche. Trouver "MyComponent" et glissez-le dans votre page. 

### Injecter du JS dans un composant

Il est important de respecter l'architecture standard sur laquelle se base AEM. Il s'attend à avoir un dossier `clientlib`  contenant un `.content.xml` (différent du composant), un dossier `css` contenant les styles, un dossier `js` contenant les fichiers Javascript et un fichier txt qui résume les fichiers js à injecter.  

1. Créer l'architecture suivante dans votre dossier mycomponent : 

 - /mycomponent
	 - /clientlib
		 - /css
			 - _mycomponent.scss
		 - /js
			 - mycomponent.js
		 - js.txt
		 - .content.xml
 - .content.xml
 - mycomponent.html

2. Remplissez le fichier xml de clientlib avec le code suivant : 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root  xmlns:cq="http://www.day.com/jcr/cq/1.0"  xmlns:jcr="http://www.jcp.org/jcr/1.0"
jcr:primaryType="cq:ClientLibraryFolder"
allowProxy="{Boolean}true"
jsProcessor="[default:none,min:none]"
categories="[wknd.mycomponent]"/>
```
> "ClientLibraryFolder" permet de servir le js que vous allez créer dans le dossier principal de librairies.
> "allowProxy" à mettre true en général, sinon votre js ne sera pas injecter
> "jsProcessor" permet d'indiquer si vous voulez minifier votre js; ici non car on souhaite le lire dans la console pour nos tests.
> "categories" équivaut au embed du xml du groupe, vous y indiquez les différents groupes et composants concerné par le js. 

3.   Le fichier `js.txt` permet de lister tous vos fichier js à utiliser. Alimentez-le ainsi : 
```txt
#base=js
mycomponent.js
```
> - Si vous organisez votre js en plusieurs fichiers, pensez à tous les lister à la suite de mycomponent.js, sinon ils ne seront pas pris en comptes. 
> 
> - "base" représente le chemin de départ.

4.  Vous pouvez alimenter votre js ainsi pour le tester : 
```js
console.log("hello");
```

5. Synchronisez votre code et activez le watch-ui. Testez votre composant en le glissant dans une page. Ouvrez la console et vérifier si votre console log apparaît. Vous pouvez sélectionner "View as publish" depuis le menu "Page information" pour être dans les conditions réelles d'affichage. 

## Boîte de dialog

On peut la styliser via la lib coral. 
Custom des field en JQuery 

### Création d'une boîte de dialogue et interaction

- Ajouter un dossier `_cq_dialog` à la racine de votre composant
- Créer un fichier `.content.xml` à l'intérieur
- Ajouter le code suivant pour avoir, par exemple, une boîte avec un champ texte : 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0" xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
    xmlns:cq="http://www.day.com/jcr/cq/1.0" xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
    jcr:primaryType="nt:unstructured"
    jcr:title="Properties"
    sling:resourceType="cq/gui/components/authoring/dialog">
    <content
        jcr:primaryType="nt:unstructured"
        sling:resourceType="granite/ui/components/coral/foundation/fixedcolumns">
        <items jcr:primaryType="nt:unstructured">
            <column
                jcr:primaryType="nt:unstructured"
                sling:resourceType="granite/ui/components/coral/foundation/container">
                <items jcr:primaryType="nt:unstructured">
                    <text
                        jcr:primaryType="nt:unstructured"
                        sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
                        fieldLabel="Text"
                        name="./text" />
                </items>
            </column>
        </items>
    </content>
</jcr:root>
```
----
  

- Vous pouvez ensuite utiliser, conditionnellement, la valeur du champ "text" rentré par l'utilisateur dans le HTML, en faisant par exemple : 
```html
  <div data-sly-test="${properties.text}">
      ${properties.text}
  </div>
```
----
 --> Vous trouverez la liste des fields et la manière de les inclure ici : https://gist.github.com/salomao-santos/0cd0240b9824b52a5fdf777ab712cfe2
 
--> Tuto avec de bonnes explications : https://satyamblogs.medium.com/granite-granite-ui-and-coral-ui-concepts-in-aem-578a11b64fcf

### Inclusion d'une dialog dans une autre dialog 

- Créer un dossier (`sharedialog` par exemple) qui contiendra toutes les boîtes de dialogues partagées et ajouter y un fichier `.content.xml` : 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:sling="http://sling.apache.org/jcr/sling/1.0" xmlns:cq="http://www.day.com/jcr/cq/1.0" xmlns:jcr="http://www.jcp.org/jcr/1.0"
    jcr:primaryType="cq:Component"
    sling:resourceSuperType="sling:folder" />
```
- Créer un sous-dossier avec le nom de votre dialog partagée, `test` par exemple, et ajouter y un fichier `.content.xml` qui contiendra les différents champs que vous souhaitez inclure dans les autres dialog (vous pouvez reprendre le xml plus haut avec le champ text) 
- Dans le xml du composant où vous souhaitez inclure votre dialog "test",  utilisez la balise `sharedialog` pour faire votre inclusion comme ceci : 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="http://www.jcp.org/jcr/1.0" xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
    xmlns:cq="http://www.day.com/jcr/cq/1.0" xmlns:sling="http://sling.apache.org/jcr/sling/1.0"
    jcr:primaryType="nt:unstructured"
    jcr:title="Properties"
    sling:resourceType="cq/gui/components/authoring/dialog">
    <content
        jcr:primaryType="nt:unstructured"
        sling:resourceType="granite/ui/components/coral/foundation/fixedcolumns">
        <items jcr:primaryType="nt:unstructured">
            <column
                jcr:primaryType="nt:unstructured"
                sling:resourceType="granite/ui/components/coral/foundation/container">
                <items jcr:primaryType="nt:unstructured">
                    <sharedialog
                        jcr:primaryType="nt:unstructured"
                        resourceType="granite/ui/components/coral/foundation/include"
                        path="/apps/wknd/components/content/sharedialog/test/content"
                    />
                </items>
            </column>
        </items>
    </content>
</jcr:root>
```
----
--> Adaptez le "path" de "sharedialog" à votre boîte de dialogue partagée.


>`/_cq_dialog` -> ce dossier indique le nœud de la boîte de dialogue
>
>Créer une boîte de dialogue générique réutilisable est une bonne pratique permettant d'éviter la duplication de code. 

Exemple inclusion boîte de dialogue avec paramètres : https://adobe-consulting-services.github.io/acs-aem-commons/features/parameterized-namespace-include/subpages/parameter-example.html

### Activer le publish

- Se rendre dans CRX
- Déplier dans l'arbre à gauche : etc/replication/agents.author/publish
- Double clic sur "publish"
- Aller sur "edit" (à côté de settings), une fenêtre apparaît 
- Sur le premier onglet "settings", cocher "enabled" et vider l'entrée "Agent User Id"
-  Dans l'onglet "transport", mettre l'url de votre publish, par exemple : `http://localhost:4503/bin/receive?sling:authRequestLogin=1` 
- Entrer le user et password dans les champs concernés (admin / admin) 
- Clic sur ok -> le toggle devrait être passé au vert
- RDV sur votre publish pour constater son fonctionnement

### OSGi

Accéder à la config OSGi : `localhost:4502/system/console/configMgr`

### Debug

Lancer AEM en mode debug (pour utiliser le debug VSCode)

`java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000 -jar aem-sdk-quickstart.jar`

adapter le port et le nom du fichier jar selon la situation
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTgyOTQwNzQ3OCwtMTU3MDU0ODU5LDEzNj
kzNzE4NTMsMjEyNjY3MzczOF19
-->