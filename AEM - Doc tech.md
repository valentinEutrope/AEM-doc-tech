---


---

<h1 id="doc-techniques">Doc techniques</h1>
<h2 id="build">Build</h2>
<h4 id="plugin">Plugin</h4>
<p>Installez le plugin"VSCode AEM Sync" sur VSCode, ce dernier permettra de synchroniser le code entre l’IDE et CRX. Entrez dans la config (@ext:yamato-ltd.vscode-aem-sync) et cochez le “Autopush” pour que le code s’importe dans AEM à chaque sauvegarde de fichier.</p>
<h4 id="mvn">MVN</h4>
<p>Les modifications java doivent se build pour apparaître dans AEM à l’instar du front qui peut s’exporter via sync. Pour build le java, il faut build le dossier core à l’aide de maven. Lancer la commande : <code>cd core &amp;&amp; mvn clean install -PautoInstallBundle</code></p>
<blockquote>
<p>A noter : vous pouvez build tout le projet (core, content, frontend, tests et apps) en lançant <code>mvn clean install -PautoInstallSinglePackage</code> directement depuis la racine du projet</p>
</blockquote>
<h2 id="composants">Composants</h2>
<p>Section traitant de la création d’un composant, de son architecture et de la manière d’y injecter du js et du style. On se base ici sur le site d’exemple WKND.</p>
<h3 id="créer-un-nouveau-composant">Créer un nouveau composant</h3>
<ol>
<li>Créer un dossier <code>mycomponent</code> dans <code>ui.apps/src/main/content/jcr_root/apps/wknd/components/content</code></li>
<li>Créer un fichier <code>mycomponent.html</code> vous pouvez y mettre le code simple suivant :</li>
</ol>
<pre class=" language-html"><code class="prism  language-html"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">class</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>mycomponent<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
    Hello
<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
</code></pre>
<ol start="3">
<li>Créer un fichier <code>.content.xml</code> qui contiendra toute les infos nécessaire pour l’éditeur AEM. Il faut définir à minima son <strong>primaryType</strong> en <code>cq:Component</code>, son <strong>title</strong>, le <strong>group</strong> auquel il appartient et un composant du core sur lequel il se base (on choisit en général un composant simple comme “container”). Vous pouvez y mettre le code exemple suivant :</li>
</ol>
<pre class=" language-xml"><code class="prism  language-xml"><span class="token prolog">&lt;?xml version="1.0" encoding="UTF-8"?&gt;</span>
<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span><span class="token namespace">jcr:</span>root</span> <span class="token attr-name"><span class="token namespace">xmlns:</span>jcr</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://www.jcp.org/jcr/1.0<span class="token punctuation">"</span></span> <span class="token attr-name"><span class="token namespace">xmlns:</span>cq</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://www.day.com/jcr/cq/1.0<span class="token punctuation">"</span></span>  <span class="token attr-name"><span class="token namespace">xmlns:</span>sling</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://sling.apache.org/jcr/sling/1.0<span class="token punctuation">"</span></span>
<span class="token attr-name"><span class="token namespace">jcr:</span>primaryType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>cq:Component<span class="token punctuation">"</span></span>
<span class="token attr-name"><span class="token namespace">jcr:</span>title</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>MyComponent<span class="token punctuation">"</span></span>
<span class="token attr-name"><span class="token namespace">sling:</span>resourceSuperType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>core/wcm/components/container/v1/container<span class="token punctuation">"</span></span>
<span class="token attr-name">componentGroup</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>WKND.Content<span class="token punctuation">"</span></span><span class="token punctuation">/&gt;</span></span>
</code></pre>
<blockquote>
<p>Notez que dans l’exemple, nous utilisons le groupe WKND, adaptez le à votre environnement au besoin.</p>
<p>A la place d’utiliser la propriété resourceSuperType pour se baser sur un composant et hériter de sa boîte de dialogue, vous pouvez créer un dossier <code>_cq_dialog</code> dans lequel vous pouvez décrire votre propre xml. Si vous ne souhaitez pas avoir de dialog, vous pouvez simplement créer le dossier vide pour que votre composant s’affiche dans la liste. Sans ce dossier ou sans un resourceType, votre composant ne s’affichera pas.</p>
</blockquote>
<ol start="4">
<li>Modifier le fichier xml global du groupe. Ce dernier liste tout les composants de votre groupe. Il faut y ajouter le votre. Par exemple, dans notre groupe WKND, nous allons nous rendre sur le fichier <code>ui.apps/src/main/content/jcr_root/apps/wknd/clientlibs/clientlib-base/.content.xml</code> et ajouter dans la liste <code>embed</code> notre composant : <code>wknd.mycomponent</code>.</li>
</ol>
<blockquote>
<p>écrire le nom du dossier et non le title du composant</p>
</blockquote>
<ol start="6">
<li>Synchronisez votre code (“export to AEM server”) et rendez-vous sur l’éditeur AEM. Sélectionner votre site ainsi qu’une page à éditer pour avoir la liste des composants à gauche. Trouver “MyComponent” et glissez-le dans votre page.</li>
</ol>
<h3 id="injecter-du-js-dans-un-composant">Injecter du JS dans un composant</h3>
<p>Il est important de respecter l’architecture standard sur laquelle se base AEM. Il s’attend à avoir un dossier <code>clientlib</code>  contenant un <code>.content.xml</code> (différent du composant), un dossier <code>css</code> contenant les styles, un dossier <code>js</code> contenant les fichiers Javascript et un fichier txt qui résume les fichiers js à injecter.</p>
<ol>
<li>Créer l’architecture suivante dans votre dossier mycomponent :</li>
</ol>
<ul>
<li>/mycomponent
<ul>
<li>/clientlib
<ul>
<li>/css
<ul>
<li>_mycomponent.scss</li>
</ul>
</li>
<li>/js
<ul>
<li>mycomponent.js</li>
</ul>
</li>
<li>js.txt</li>
<li>.content.xml</li>
</ul>
</li>
</ul>
</li>
<li>.content.xml</li>
<li>mycomponent.html</li>
</ul>
<ol start="2">
<li>Remplissez le fichier xml de clientlib avec le code suivant :</li>
</ol>
<pre class=" language-xml"><code class="prism  language-xml"><span class="token prolog">&lt;?xml version="1.0" encoding="UTF-8"?&gt;</span>
<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span><span class="token namespace">jcr:</span>root</span>  <span class="token attr-name"><span class="token namespace">xmlns:</span>cq</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://www.day.com/jcr/cq/1.0<span class="token punctuation">"</span></span>  <span class="token attr-name"><span class="token namespace">xmlns:</span>jcr</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://www.jcp.org/jcr/1.0<span class="token punctuation">"</span></span>
<span class="token attr-name"><span class="token namespace">jcr:</span>primaryType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>cq:ClientLibraryFolder<span class="token punctuation">"</span></span>
<span class="token attr-name">allowProxy</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>{Boolean}true<span class="token punctuation">"</span></span>
<span class="token attr-name">jsProcessor</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>[default:none,min:none]<span class="token punctuation">"</span></span>
<span class="token attr-name">categories</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>[wknd.mycomponent]<span class="token punctuation">"</span></span><span class="token punctuation">/&gt;</span></span>
</code></pre>
<blockquote>
<p>“ClientLibraryFolder” permet de servir le js que vous allez créer dans le dossier principal de librairies.<br>
“allowProxy” à mettre true en général, sinon votre js ne sera pas injecter<br>
“jsProcessor” permet d’indiquer si vous voulez minifier votre js; ici non car on souhaite le lire dans la console pour nos tests.<br>
“categories” équivaut au embed du xml du groupe, vous y indiquez les différents groupes et composants concerné par le js.</p>
</blockquote>
<ol start="3">
<li>Le fichier <code>js.txt</code> permet de lister tous vos fichier js à utiliser. Alimentez-le ainsi :</li>
</ol>
<pre class=" language-txt"><code class="prism  language-txt">#base=js
mycomponent.js
</code></pre>
<blockquote>
<ul>
<li>
<p>Si vous organisez votre js en plusieurs fichiers, pensez à tous les lister à la suite de mycomponent.js, sinon ils ne seront pas pris en comptes.</p>
</li>
<li>
<p>“base” représente le chemin de départ.</p>
</li>
</ul>
</blockquote>
<ol start="4">
<li>Vous pouvez alimenter votre js ainsi pour le tester :</li>
</ol>
<pre class=" language-js"><code class="prism  language-js">console<span class="token punctuation">.</span><span class="token function">log</span><span class="token punctuation">(</span><span class="token string">"hello"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<ol start="5">
<li>Synchronisez votre code et activez le watch-ui. Testez votre composant en le glissant dans une page. Ouvrez la console et vérifier si votre console log apparaît. Vous pouvez sélectionner “View as publish” depuis le menu “Page information” pour être dans les conditions réelles d’affichage.</li>
</ol>
<h2 id="boîte-de-dialog">Boîte de dialog</h2>
<p>On peut la styliser via la lib coral.<br>
Custom des field en JQuery</p>
<h3 id="création-dune-boîte-de-dialogue-et-interaction">Création d’une boîte de dialogue et interaction</h3>
<ul>
<li>Ajouter un dossier <code>_cq_dialog</code> à la racine de votre composant</li>
<li>Créer un fichier <code>.content.xml</code> à l’intérieur</li>
<li>Ajouter le code suivant pour avoir, par exemple, une boîte avec un champ texte :</li>
</ul>
<pre class=" language-xml"><code class="prism  language-xml"><span class="token prolog">&lt;?xml version="1.0" encoding="UTF-8"?&gt;</span>
<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span><span class="token namespace">jcr:</span>root</span> <span class="token attr-name"><span class="token namespace">xmlns:</span>jcr</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://www.jcp.org/jcr/1.0<span class="token punctuation">"</span></span> <span class="token attr-name"><span class="token namespace">xmlns:</span>nt</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://www.jcp.org/jcr/nt/1.0<span class="token punctuation">"</span></span>
    <span class="token attr-name"><span class="token namespace">xmlns:</span>cq</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://www.day.com/jcr/cq/1.0<span class="token punctuation">"</span></span> <span class="token attr-name"><span class="token namespace">xmlns:</span>sling</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://sling.apache.org/jcr/sling/1.0<span class="token punctuation">"</span></span>
    <span class="token attr-name"><span class="token namespace">jcr:</span>primaryType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>nt:unstructured<span class="token punctuation">"</span></span>
    <span class="token attr-name"><span class="token namespace">jcr:</span>title</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>Properties<span class="token punctuation">"</span></span>
    <span class="token attr-name"><span class="token namespace">sling:</span>resourceType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>cq/gui/components/authoring/dialog<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>content</span>
        <span class="token attr-name"><span class="token namespace">jcr:</span>primaryType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>nt:unstructured<span class="token punctuation">"</span></span>
        <span class="token attr-name"><span class="token namespace">sling:</span>resourceType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>granite/ui/components/coral/foundation/fixedcolumns<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>items</span> <span class="token attr-name"><span class="token namespace">jcr:</span>primaryType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>nt:unstructured<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>column</span>
                <span class="token attr-name"><span class="token namespace">jcr:</span>primaryType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>nt:unstructured<span class="token punctuation">"</span></span>
                <span class="token attr-name"><span class="token namespace">sling:</span>resourceType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>granite/ui/components/coral/foundation/container<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>items</span> <span class="token attr-name"><span class="token namespace">jcr:</span>primaryType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>nt:unstructured<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>text</span>
                        <span class="token attr-name"><span class="token namespace">jcr:</span>primaryType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>nt:unstructured<span class="token punctuation">"</span></span>
                        <span class="token attr-name"><span class="token namespace">sling:</span>resourceType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>granite/ui/components/coral/foundation/form/textfield<span class="token punctuation">"</span></span>
                        <span class="token attr-name">fieldLabel</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>Text<span class="token punctuation">"</span></span>
                        <span class="token attr-name">name</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>./text<span class="token punctuation">"</span></span> <span class="token punctuation">/&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>items</span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>column</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>items</span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>content</span><span class="token punctuation">&gt;</span></span>
<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span><span class="token namespace">jcr:</span>root</span><span class="token punctuation">&gt;</span></span>
</code></pre>
<hr>
<ul>
<li>Vous pouvez ensuite utiliser, conditionnellement, la valeur du champ “text” rentré par l’utilisateur dans le HTML, en faisant par exemple :</li>
</ul>
<pre class=" language-html"><code class="prism  language-html">  <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>div</span> <span class="token attr-name">data-sly-test</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>${properties.text}<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
      ${properties.text}
  <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>div</span><span class="token punctuation">&gt;</span></span>
</code></pre>
<hr>
<p>–&gt; Vous trouverez la liste des fields et la manière de les inclure ici : <a href="https://gist.github.com/salomao-santos/0cd0240b9824b52a5fdf777ab712cfe2">https://gist.github.com/salomao-santos/0cd0240b9824b52a5fdf777ab712cfe2</a></p>
<p>–&gt; Tuto avec de bonnes explications : <a href="https://satyamblogs.medium.com/granite-granite-ui-and-coral-ui-concepts-in-aem-578a11b64fcf">https://satyamblogs.medium.com/granite-granite-ui-and-coral-ui-concepts-in-aem-578a11b64fcf</a></p>
<h3 id="inclusion-dune-dialog-dans-une-autre-dialog">Inclusion d’une dialog dans une autre dialog</h3>
<ul>
<li>Créer un dossier (<code>sharedialog</code> par exemple) qui contiendra toutes les boîtes de dialogues partagées et ajouter y un fichier <code>.content.xml</code> :</li>
</ul>
<pre class=" language-xml"><code class="prism  language-xml"><span class="token prolog">&lt;?xml version="1.0" encoding="UTF-8"?&gt;</span>
<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span><span class="token namespace">jcr:</span>root</span> <span class="token attr-name"><span class="token namespace">xmlns:</span>sling</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://sling.apache.org/jcr/sling/1.0<span class="token punctuation">"</span></span> <span class="token attr-name"><span class="token namespace">xmlns:</span>cq</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://www.day.com/jcr/cq/1.0<span class="token punctuation">"</span></span> <span class="token attr-name"><span class="token namespace">xmlns:</span>jcr</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://www.jcp.org/jcr/1.0<span class="token punctuation">"</span></span>
    <span class="token attr-name"><span class="token namespace">jcr:</span>primaryType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>cq:Component<span class="token punctuation">"</span></span>
    <span class="token attr-name"><span class="token namespace">sling:</span>resourceSuperType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>sling:folder<span class="token punctuation">"</span></span> <span class="token punctuation">/&gt;</span></span>
</code></pre>
<ul>
<li>Créer un sous-dossier avec le nom de votre dialog partagée, <code>test</code> par exemple, et ajouter y un fichier <code>.content.xml</code> qui contiendra les différents champs que vous souhaitez inclure dans les autres dialog (vous pouvez reprendre le xml plus haut avec le champ text)</li>
<li>Dans le xml du composant où vous souhaitez inclure votre dialog “test”,  utilisez la balise <code>sharedialog</code> pour faire votre inclusion comme ceci :</li>
</ul>
<pre class=" language-xml"><code class="prism  language-xml"><span class="token prolog">&lt;?xml version="1.0" encoding="UTF-8"?&gt;</span>
<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span><span class="token namespace">jcr:</span>root</span> <span class="token attr-name"><span class="token namespace">xmlns:</span>jcr</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://www.jcp.org/jcr/1.0<span class="token punctuation">"</span></span> <span class="token attr-name"><span class="token namespace">xmlns:</span>nt</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://www.jcp.org/jcr/nt/1.0<span class="token punctuation">"</span></span>
    <span class="token attr-name"><span class="token namespace">xmlns:</span>cq</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://www.day.com/jcr/cq/1.0<span class="token punctuation">"</span></span> <span class="token attr-name"><span class="token namespace">xmlns:</span>sling</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>http://sling.apache.org/jcr/sling/1.0<span class="token punctuation">"</span></span>
    <span class="token attr-name"><span class="token namespace">jcr:</span>primaryType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>nt:unstructured<span class="token punctuation">"</span></span>
    <span class="token attr-name"><span class="token namespace">jcr:</span>title</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>Properties<span class="token punctuation">"</span></span>
    <span class="token attr-name"><span class="token namespace">sling:</span>resourceType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>cq/gui/components/authoring/dialog<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>content</span>
        <span class="token attr-name"><span class="token namespace">jcr:</span>primaryType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>nt:unstructured<span class="token punctuation">"</span></span>
        <span class="token attr-name"><span class="token namespace">sling:</span>resourceType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>granite/ui/components/coral/foundation/fixedcolumns<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>items</span> <span class="token attr-name"><span class="token namespace">jcr:</span>primaryType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>nt:unstructured<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>column</span>
                <span class="token attr-name"><span class="token namespace">jcr:</span>primaryType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>nt:unstructured<span class="token punctuation">"</span></span>
                <span class="token attr-name"><span class="token namespace">sling:</span>resourceType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>granite/ui/components/coral/foundation/container<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>items</span> <span class="token attr-name"><span class="token namespace">jcr:</span>primaryType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>nt:unstructured<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span>
                    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>sharedialog</span>
                        <span class="token attr-name"><span class="token namespace">jcr:</span>primaryType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>nt:unstructured<span class="token punctuation">"</span></span>
                        <span class="token attr-name">resourceType</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>granite/ui/components/coral/foundation/include<span class="token punctuation">"</span></span>
                        <span class="token attr-name">path</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>/apps/wknd/components/content/sharedialog/test/content<span class="token punctuation">"</span></span>
                    <span class="token punctuation">/&gt;</span></span>
                <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>items</span><span class="token punctuation">&gt;</span></span>
            <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>column</span><span class="token punctuation">&gt;</span></span>
        <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>items</span><span class="token punctuation">&gt;</span></span>
    <span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>content</span><span class="token punctuation">&gt;</span></span>
<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span><span class="token namespace">jcr:</span>root</span><span class="token punctuation">&gt;</span></span>
</code></pre>
<hr>
<p>–&gt; Adaptez le “path” de “sharedialog” à votre boîte de dialogue partagée.</p>
<blockquote>
<p><code>/_cq_dialog</code> -&gt; ce dossier indique le nœud de la boîte de dialogue</p>
<p>Créer une boîte de dialogue générique réutilisable est une bonne pratique permettant d’éviter la duplication de code.</p>
</blockquote>
<p>Exemple inclusion boîte de dialogue avec paramètres : <a href="https://adobe-consulting-services.github.io/acs-aem-commons/features/parameterized-namespace-include/subpages/parameter-example.html">https://adobe-consulting-services.github.io/acs-aem-commons/features/parameterized-namespace-include/subpages/parameter-example.html</a></p>

