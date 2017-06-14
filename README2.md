# Coopgui

Coopgui is a php graphics library which allows you to build the front end part of your website very precisely by generating HTML, CSS and JavaScript.
Coopgui reduces the use of Javascript for security reasons. For example the HTML elements are generated using PHP and not JavaScript.

## Prerequisites
A server with php.

## Basic Installation
* Copy everything on your server.
* Make sure all requests are directed to the [index.php](https://github.com/coopattitude/coopgui/blob/master/index.php) file by your web server configuration.
* Each time you change a file that generates CSS or JavaScript you have to delete the files in the [gen] folder. This forces coopgui to recreate these gen files and therefore allows the changes to be effective. __(?)pourquoi mettre cette phrase dans la partie installation ? Est-ce que ça a un rapport avec l'installation ?

## coopgui arborescence 
this library works like this: to build a website the user creates daughter classes in [site](https://github.com/coopattitude/coopgui/tree/master/site) and [plugins](https://github.com/coopattitude/coopgui/tree/master/plugins). This classes inherit from the classes inside the system folder. __(moi je trouvais ça important, pourquoi l'enlever ?)__ 


system : the classes that the user can reuse (=create a daughter class)
plugins:
gen:
js:

FB:
vérifier classe fils/fille
réécrire
pas de détail
le site appel desplugins qui appelle d'autre pollugins et ça permet de monter uns site web
nested plugin explication conceptuellement (détaille plus bas ds #nestedplugin)

## How coopgui works

### HTML generation overview
(The core of the generation is the plugins)
Coopgui generates the HTML of a web page by calling plugins located in the [plugins](https://github.com/coopattitude/coopgui/tree/master/plugins) directory. (Therefore) to create a web page you need to slice it in several parts and write a plugin for each part. Then specify the plugins that will be called/ to use/ instantiate those (?) plugins in the [site/index.php] file. 
When a web browser sends a request, this [site/index.php](https://github.com/coopattitude/coopgui/blob/master/site/index.php) file, which is the main file of the site, is executed. This file successively calls all the plugins (it needs) and pass them options. Each plugin returns a string containing HTML to the index.php file which then appends everything into an array. Eventually the array is sent using echo. 

A very important feature is that the plugins can be nested. For example when a plugin is called it calls __(is it the correct tense after when ?)__ another plugin. The latter returns his string containing HTML to the first which then appends it into his own html string and then returns it to the index.php file. You can nest as many plugins you want inside a plugin. Those plugins that are nested can themselves call as many plugins they want which themselves call as many plugins they want, etc. It is unlimited. This allows you do build a more complicated __(trouver autre mot)__ web page than just appending plugins one after the other. __(dernière phrase à reformuler)__





## site/index.php
The [site/index.php](https://github.com/coopattitude/coopgui/blob/master/site/index.php) file is the main file of the site. It is the first file to be executed and depending on the request it generates the appropriate web page by calling specific plugins.
This file defines the class `IrIndex` and immediately creates an IrIndex object (and then calls his init method). 

### The constructor
In the constructor, you have to instantiate these 4 classes like this :
```php
$this->irCommander = new \IrCommander ($this->irCommander) ;
$this->irEnvCustom = new \IrSystemEnvCustom ($this->irCommander) ;
$this->irGenCss = new \IrSystemGenCss ($this->irCommander, $this->irEnvCustom) ;
$this->irGenJs = new \IrSystemGenJs ($this->irCommander, $this->irEnvCustom) ;
```
__(?)pourquoi on envoie l'objet irCommander au constructeur alors qu'il est même pas créé encore ?(/?)__ 

The [IrCommander] object allows you to... __(?)j'ai pas bien compris à quoi il servait. Il contient des variables qui indiquent s'il faut refaire le cache et des variables sur la version css et js. Il instancie IrGlobalVar et créé la fonction g pour que on puisse utiliser cet objet partout. Du coup je vois pas trop quelle est sa fonction(/?)__

The [IrSystemEnvCustom] object allows you to store data to customize specific properties of the web page __(the envionnement ?)__. For example the color. __(?)oui mais c la couleur de quoi ? (/?)__ You have to give this object to the `IrGenCss` and `IrGenJS` objects so that they can take into account these custom properties. 
__(?)dois-je préciser que ces quatres objets sont passé très souvent quand on créé un objet ?(/?)

The [IrSystemGenCss] and [IrSystemGenJs] objects will be used to generate the CSS and the JavaScript. See the cache section for more details.
(__insérer un lien vers l' encre de la section cache__)


LC:
personnalisation du cache -> fait
personnalisation de l'environnement -> je ne vois pas quoi mettre dedans

# HTML generation
This IrIndex object that was just created __(quel temps utiliser ?)__ echos the HTML of the web page. The head method generates the head and the init method generates the body. 

IrIndex echos the HTML only at the end. This avoids sending half the page when an error occurs. In order to do that you have to append all the HTML strings in one array. This array is the Irindex propertie `display` and is echoed at the end of the init method.

### head method
In the `head` method you can append everything you want to be in the head tag : meta tags, title tags, etc. You can also append link and script tags to include the stylesheets and the JavaScript files you want. __(?)Dire que Jquery est ds la bibliothèque et que on peut l'inclure si on veut?(/?)__ __(?)Dire que on peut inclure une police ? La police Roboto ?(/?)__

FB:
le jquery et la police roboto sont utiliser de base par défaut
Ne pas expliquer comment changer ou rajouter une police.

You also have to call the `initCss` method on the IrIndex propertie `irGenCss` and append the result into `display` like this :
```php
$linkCss = $this->irGenCss->initCss () ;
$this->display [] = $linkCss ;
```
This gathers the css of all the plugins in one file and returns a link tag with the path to this file. 

Do the same with the `initJs` method on the IrIndex propertie `irGenJs`. This gathers all the JavaScript file inside the [plugins](https://github.com/coopattitude/coopgui/tree/master/plugins) and the js [folder](https://github.com/coopattitude/coopgui/tree/master/js) into one file and returns a script tag that imports it.


### init method
In the `init` method you can append everything you want to go in the body tag. Instantiate all the plugin objects you need to build your web page. Call their `toHtml` method on them and append the result in `display`. For example for the Info plugin :

```php
$irHtmlInfo = new \IrHtmlInfo ($this->irCommander, $this->irGenCss) ;
$this->display [] = $irHtmlInfo->toHtml () ;
```

At the end of the init don't forget to close the body and html tags and then echo everything that's inside `display` like this:

```php
$this->display [] = '</body>' ;
$this->display [] = '</html>' ;

echo (implode ('', $this->display)) ;
```

## Plugins
For coopgui to generate a web page you have to write plugins. A plugin represents a part of the page. For example a clock could be a plugin. The plugins must be inside the [plugins](https://github.com/coopattitude/coopgui/tree/master/plugins) directory and contain a PHP file for the HTML generation, a PHP file for the CSS generation and a JavaScript file that contains the code related to the plugin.

#### How to create a Plugin
Create a folder with the plugin name in the [plugins](https://github.com/coopattitude/coopgui/tree/master/plugins) directory.
##### HTML generation
The plugin's HTML is generated by a PHP class. Below is how to create one. 
* Inside the plugin folder, create a php file with the plugin name. For example [IrHtmlInfo.php](https://github.com/coopattitude/coopgui/blob/master/plugins/Info/IrHtmlInfo.php) for the info plugin.
* Inside this file, define a php class with the same name as the file (the file and the class must have the same name for the autoloader/classloader to work correctly). This class must inherit from the IrHtmlFather class which is located in the [system/Html](https://github.com/coopattitude/coopgui/tree/master/system/Html) directory.
* Add the class to the classloader [IrGuiClassLoader.php](https://github.com/coopattitude/coopgui/blob/master/system/ClassLoader/IrGUIClassLoader.php) as explained in the classloader section of this documentation.
* Create a method `toHtml`. This is the method which will return the HTML of the plugin.
* Inside this method, create an array called "$htmlDef". 

__(?)Expliquer le unset ?(/?)__
FB:
pas évident ça vide le tableau 
prend la def php de unset
LC: expliquer plus en commentaire peut être ?

Fill it with strings that represents the HTML tags you want to return. For example in IrHtmlInfo.php :

  __(?)faire une colonne example à droite ? genre tableau(/?)__
FB:
mettre colonne à droite avec le code html rendu
  ```php
  unset ($htmlDef) ;
		
  $htmlDef [] = '<div[htmlInfoBox]>' ;

    $htmlDef [] = '<div[htmlInfoBoxLeft]>' ;
      $htmlDef [] = '<div[htmlInfoBoxLeftImg]></div>' ;
    $htmlDef [] = '</div>' ;

    $htmlDef [] = '<div[htmlInfoBoxRight]>' ;

      $htmlDef [] = '<div[htmlInfoBoxRightTop]>' ;

        $htmlDef [] = '<div[htmlInfoBoxRightTopTitle]></div>' ;
        $htmlDef [] = '<div[htmlInfoBoxRightTopTop]></div>' ;
        $htmlDef [] = '<div[htmlInfoBoxRightTopMiddle]></div>' ;
        $htmlDef [] = '<div[htmlInfoBoxRightTopBottom]></div>' ;

      $htmlDef [] = '</div>' ;

    $htmlDef [] = '</div>' ;

  $htmlDef [] = '</div>' ;
  ```
	```html
	<div class="css1_htmlInfoBox" id="htmlInfoBox_prefix_IrHtmlInfo_1_r_62638" >
		blah blah blah
		<div class="css1_htmlInfoBoxLeft" >
			<div class="css1_htmlInfoBoxLeftImg" >
			</div>
		</div>
		<div class="css1_htmlInfoBoxRight" 	>
			<div class="css1_htmlInfoBoxRightTop" >
				<div class="css1_htmlInfoBoxRightTopTitle" >
				</div>
				<div class="css1_htmlInfoBoxRightTopTop" >
				</div>
				<div class="css1_htmlInfoBoxRightTopMiddle" >
				</div>
				<div class="css1_htmlInfoBoxRightTopBottom" >
				</div>
			</div>
		</div>
	</div>
	```

  The strings inside the square brackets are flags you give to the tags in order to retrieve them later in the code.

* Create an IrHtmlDef object and pass it the htmlDef array. For example in IrHtmlInfo.php :
  ```php
  $irHtmlDef = new \IrHtmlDef ($this->irCommander, $this->getIrGenCss (), $htmlDef) ;
  ```

  This [irHtmlDef](https://github.com/coopattitude/coopgui/blob/master/system/Lib/IrHtmlDef.php) object will scan $htmlDef, find the tags and create one [IrHtmlTag](https://github.com/coopattitude/coopgui/blob/master/system/Lib/IrHtmlTag.php) object per tag. 

* Modifiy each tag as you want. First retrieve the wanted tag using his flag and the get method of the irHtmlDef object like this :

  ```php
  $irHtmlTag = $irHtmlDef->get ('htmlInfoBox') ;
  ```
 Then you can modifiy this specific tag using IrHtmlTag methods. For example you can give it a class, an id and a content :

  ```php
  $irHtmlTag->setClass ('htmlInfoBox') ;
  $irHtmlTag->setId ('htmlInfoBox_'.$this->getPrefixUniq ()) ;
  $irHtmlTag->appendContent('blah blah blah');
  ```
  
  In this case the class is the same as the flag but it is a coincidence. The flag is just a label to retrieve the tag. 
  
* At the end of the toHtml() method, return what the toHtml method of the IrHtmlDef object returns like this :
  ```php
  return $irHtmlDef->toHtml ()
  ```
This toHtml method browses `$htmlDef` and for each tag it replaces the pair of square brackets with the attributes that were given to the corresponding tag. Then it returns it.

# scriptInit
## prefixUniq
Each time you create a plugin object, Coopgui generates a prefixUniq for this object. This prefixUniq is like an ID and it is unique. No other plugin will have the same. For example one possible prefixUniq for the first Info plugin instance is : 
`'prefix_IrHtmlInfo_1_r_62638'`
It is generated like this :
`prefix + nom de la classe + numéro de création de l'instance + r + nombre aléatoire entre 0 et 1 million.`
This makes sure it is unique __(?)mais pourquoi un nombre aléatoire. Le numéro de l'instance ne suffit pas ? Tu m'avais dit
que ça ne suffisait pas à cause des appels ajax. Dois-je le préciser ?(/?)__

This prefixUniq allows the JavaScript to retrieve the tags of this specific plugin in the DOM. __(?)Besoin de préciser l'abréviation ?(/?)__

__(?)pourquoi ça s'appelle un préfix ? pourtant on le met après un mot ça devrait donc s'appeler un suffix non ? (?)__

### how to use it
#### labelling
When you create the tags of your plugin in your plugin's html generation class in the `toHtml` method, create a tag that includes all the other. Use the `setId` method on this tag to set his id attribut value and append the prefixUniq to this value. For example for the Info plugin :

```php
$irHtmlTag = $irHtmlDef->get ('htmlInfoBox') ;  //retrieves the tag that contains all the other
$irHtmlTag->setClass ('htmlInfoBox') ;
$irHtmlTag->setId ('htmlInfoBox_'.$this->getPrefixUniq ()) ;  //sets the id and appends the prefixUniq to the id value
$irHtmlTag->appendContent('blah blah blah');
```
To include the prefixUniq, use the `getPrefixUniq` method of the plugin instance.

#### JavaScript selection
Now in the JavaScript file if you want to select this specific plugin in the DOM, use an id selector and specify the prefixUniq. For example for the Info plugin in the [IrHtmlInfo.js] file : 

```javascript
$('#htmlInfoBox_'+prefixUniq).width ()
```
__(?)préciser que c du jquery ?et si oui comment préciser ?(/?)__

# scriptInit
__(transition depuis prefixUniq ?)__
The problem is that the JavaScript doesn't know the prefixUniq because it was generated by the PHP. You need a means to give data/informations __(which one ?)__ to the JavaScript. That is what the `scriptInit` method is for.
`scriptInit` is a method of your plugin instance. `scriptIni` allows you to pass data/informations __(which one ?)__ to the JavaScript.
Find the return satement in the `toHtml` method of the plugin's html generation class. Call `scriptInit` and pass it the name of the plugin and the data/informations __(which one ?)__ you want to send to the JavaScript. Then (,) append the result of `scriptInit` to what is already returned. For example in the Info plugin :

```php
return $irHtmlDef->toHtml ().$this->scriptInit ('htmlInfo', $this->getPrefixUniq (), $this->getIrGenCss ()->css) ;
```
Note that 
`scriptInit` returns a JavaScript script tag which calls the "init" JavaScript function of the plugin and pass it the data/informations __(which one ?)__ `scriptInit` received. For example in the Info plugin the call to `scriptInit` in the block of code above will return this : 

```html
<script type="text/javascript">
	$(function() {
			htmlInfoInit ('prefix_IrHtmlInfo_1_r_62638', 'css1');
	});
</script>
```
Here the data/informations __(which one ?)__ we send is the prefixUniq of the plugin and the css reference to the JavaScript.
The `htmlInfoInit` function is the "init" JavaScript function of the plugin. It must be defined in the JavaScript file of the plugin.

Because the informations are sent via a script tag you can only pass strings. __(?)Comment le JavaScript reconnait l'encodage des chaines de caractères qui sont envoyés par htmlInfoInit ?(/?)__

# nested plugins
A very important feature is that the plugins can be nested. For example when a plugin is called it calls __(is it the correct tense after when ?)__ another plugin. The latter returns his string containing HTML to the first which then appends it into his own html string and then returns it to the index.php file. You can nest as many plugins you want inside a plugin. Those plugins that are nested can themselves call as many plugins they want which themselves call as many plugins they want, etc. It is unlimited. This allows you do build a more complicated web page than just appending plugins one after the other.

## how to nest a plugin inside another one
For example, let's assume you want to nest plugin2 inside plugin1 knowing that plugin2 is already fully written in the [plugin] folder. In plugin1's html generation class in the `toHtml` method write :

```php
$htmlDef [] = '<div[father]>' ;
             *
             *
	/*Some tags*/
	     *
	     *
	$htmlDef [] = '<div[plugin2]></div>' ; //create the tag that will contain the result of plugin2
	     *
             *
	/*Some tags*/
	     *
	     *
$htmlDef [] = '</div>' ;

$irHtmlDef = new \IrHtmlDef ($this->irCommander, $this->getIrGenCss (), $htmlDef) ;

//Create a Plugin2 object.
$irPlugin2 = new IrPlugin2($this->irCommander, $this->irGenCss) ;

//retrieve the tag that will contain the result of plugin2.
$irHtmlTag = $irHtmlDef->get ('plugin2') ;  

//call the toHtml method on the Plugin2 object and append the result into the tag's content.
$irHtmlTag->appendContent($irPlugin2->toHtml()); 
```



##### CSS generation
A plugin's CSS style sheet is generated by a PHP class. Below is how to create one.
This class have to generate CSS that applies on the tags definied above in the HTML generation class

* Inside the plugin folder, create a folder called "css" and then inside a php file. To name this php file, concatenate "IrCss_" with the plugin name. For example the CSS generation file of the Info plugin is [IrCSS_HtmlInfo.php](https://github.com/coopattitude/coopgui/blob/master/plugins/Info/css/IrCss_htmlInfo.php). This "IrCss_" is very important as it indicates to [IrLibGenCss](https://github.com/coopattitude/coopgui/blob/master/system/Lib/IrLibGenCss.php) which files to collect for the CSS style sheet generation.
* Inside this file, define a php class with the same name as the file. This class must inherit from the `IrCssFather` class which is located in the [system/Css](https://github.com/coopattitude/coopgui/tree/master/system/Css) directory.
* redefine the constructor and inside call the parent constructor and then call the `init` method
* in the `init` method write code to generate all the CSS rule-sets you want

###### how to generate a rule-set
The [system/Css](https://github.com/coopattitude/coopgui/tree/master/system/Css) repository contains classes called IrCssClassDiv, IrCssClassBody, etc. These classes are used to generate CSS rule-sets. 
For example the [IrHtmlInfo](https://github.com/coopattitude/coopgui/blob/master/plugins/Info/IrHtmlInfo.php) class of the info plugin generates a div tag with the class `htmlInfoBox`. The [IrCSS_HtmlInfo](https://github.com/coopattitude/coopgui/blob/master/plugins/Info/css/IrCss_htmlInfo.php) class generates a CSS rule-set that applies on this div tag. Below is how this rule-set is created:

```php
$irCssClass = $this->add (new IrCssClassDiv ('htmlInfoBox')) ;
$irCssClass->setDisplay ('block') ;
$irCssClass->setWidthPercent (100) ;
$irCssClass->setTextAlign ('center') ;
$irCssClass->setBoxSizing ('border-box') ;

$irCssClass->setBackgroundColor ('#f5f9fc') ;
```
Here are the steps:
* create a IrCssClassTag object (where "Tag" should be replaced by the correct tag) and pass it the name of the class.
* pass this object to the `add` method of the [IrCSS_HtmlInfo](https://github.com/coopattitude/coopgui/blob/master/plugins/Info/css/IrCss_htmlInfo.php) class. This method registers the object and returns it.
* use the methods availabe on IrCssClassTag objects to add the CSS properties you want and their corresponding value

This code will generate this in the CSS style sheet :

```Css
div.css1_htmlInfoBox {
    text-align: center;
    background-color: #f5f9fc;
    display: block;
    width: 100%;
    box-sizing: border-box;
    -webkit-box-sizing: border-box;

    margin: 0px 0px 0px 0px;
    padding: 0px 0px 0px 0px;
    font-size: 14.4px;
    font-size: 1.44rem;
    line-height: 1.98rem;

}
```

There are more CSS properties because some of them are default properties inherited from the IrCssClassDiv or the IrCssClassFather classes. IrCssClassFather is the father of all the IrCssClassTag classes. If the IrCssClassTag you want doesn't exist you can create your own IrCssClassTag or use IrCssClassFather directly.
Unlike the HTML generation file, this class doesn't need to be added to any ClassLoader class because it is included manually.

###### style sheet generation
There is one CSS style sheet for all the plugins. When index.php receives a request, the IrLibGenCss class collects the CSS classes of all the plugins, use them to generate CSS and appends everything into the file css1_all.css in gen/css

###### responsive design (css part)
FB : juste partie 
le fichier responsive dans coopgui : pas besoin de commentaire ou peut être si au début

###### inline CSS ?
en parler au niveau de l'htmldef plus haut
c la méthode appendIrHtmlatt (att comme attribut) qui permet ça (après avoir créé un objet IrHtmlAtt)



##### plugin's JavaScript (trouver un meilleur titre)
Inside the plugin folder create a javascript file. For example [IrHtmlInfo.js]()
dire que c du JS classique mais facilité graĉe au du préfixuniq
jquery classique
préfix uniq = moyen de communiquer enre php et JS comme dit plus haut 
mettre une encre pour renvoyer
encre = balise a avec href= #nombalised'encre


__(mettre une ancre pr le cache)__
## Cache
In the [IrIndex] class constructor (see the index.php section) __(insérer une ancre)__ we create an [IrSystemGenCss] and an [IrSystemGenJs] object. The [IrSystemGenCss] object will be used to generate the CSS of all the plugins and gather them into one file. The [IrSystemGenJs] object will be used to gather all the JavaScript files inside the [plugins](https://github.com/coopattitude/coopgui/tree/master/plugins) and the js [folder](https://github.com/coopattitude/coopgui/tree/master/js) into one file.

Those two files are cached in the gen folder. This means they are generated only the first time [index.php] receives a request. This is to avoid generating the same files each time a request is sent. The downside is that each time you change a JavaScript file or a file that generates CSS you have to delete the files in the [gen] folder for the changes to be effetive. This forces coopgui to recreate these gen files and therefore takes the changes into account.

### Cache personalization
* You can force the cache to be generated every time a request is sent. In the IrIndex constructor after creating an [IrCommander] object, change the IrCommander properties `boolRedoJsCache` and `boolRedoCssCache` to true like this: 
  ```php
  $this->irCommander = new \IrCommander ($this->irCommander) ;
  $this->irCommander->boolRedoJsCache = true;
  $this->irCommander->boolRedoCssCache = true;
  ```
  The `boolRedoJsCache` propertie is for the JavaScript and the `boolRedoCssCache` for the CSS.

* You can customize the path were the cache files are generated:
    * For the CSS.
				By default the CSS file is generated in _gen/css_ . Suppose that you want the generation file to become _site1/gen/css_ . In [site/IrSystemGenCss] in the `IrSystemGenCss` constructor, assign to the `prefixGenPath` propertie the string containing 'site1'.
   
    * For the JavaScript:
				The method is the same as the CSS. The only difference is that you have to go in [site/IrSystemGenJs] in the `IrSystemGenJs` constructor.

__(?)Par rapport au cache : Comment avoir plusieurs site avec coopgui ? avoir deux dossier gen ?(/?)__



# ClassLoader
Coopgui containes two classloader classes : [IrGuiClassLoader](https://github.com/coopattitude/coopgui/blob/master/system/ClassLoader/IrGUIClassLoader.php) and [ClassLoader](https://github.com/coopattitude/coopgui/blob/master/site/ClassLoader.php). These classes allows you to use all the other classes whithout having to include them in each file. The first class takes care of the classes inside [system](https://github.com/coopattitude/coopgui/tree/master/system) and [plugins](https://github.com/coopattitude/coopgui/tree/master/plugins). The second class takes care of the classes inside [site](https://github.com/coopattitude/coopgui/tree/master/site). 
The classloader classes are themselves included at the beginning of the [site/index.php](https://github.com/coopattitude/coopgui/blob/master/site/index.php) file

## How to add a class to a classloader class
Each time you create a new plugin or a new class inside [site](https://github.com/coopattitude/coopgui/tree/master/site) you have to add the class to the correct classloader class.
For example let's assume you created a plugin called "MyPlugin" with a IrMyplugin.php file in it (which itself defines the IrMyPlugin class) : 
* Open [IrGuiClassLoader.php](https://github.com/coopattitude/coopgui/blob/master/system/ClassLoader/IrGUIClassLoader.php) 
* Inside the loader method find the switch statement that looks like this :
  ```php
  switch ($className) {

    case 'IrContainer' :
    case 'IrHtmlAtt' :
    case 'IrHtmlDef' :
    case 'IrHtmlTag' :
    case 'IrLibEnvCustom' :
    case 'IrLibGenCss' :
    case 'IrLibGenJs' :
      $filePath = 'system/Lib/'.$className.'.php' ;
      break ;

    .
    .
    .

    case 'IrHtmlInfo' :
      $filePath = 'plugins/Info/'.$className.'.php' ;
      break ;
  ```
* Add a case for when $className contains "IrMyPlugin" like this :
  ```php
    case 'IrMyPlugin' :
      $filePath = 'plugins/MyPlugin/'.$className.'.php' ;
      break ;
  ```
  
The process is the same with the [ClassLoader](https://github.com/coopattitude/coopgui/blob/master/site/ClassLoader.php) class.

#### CSS Generation
FB: pas besoin de détaillé le moteur de la génération du CSS et du JS

### How to create a website
To create a website you need to slice it in several plugins. Plus exactement. You have to glue plugins together, to nest them Each plugin represents a part of the site. The [site/index.php](https://github.com/coopattitude/coopgui/blob/master/site/index.php) file   in the site folder coopgui allows you to create plugins in the plugin folder. The site folder  

### Example site

example site htmlInfo, horloge
FB: 
résumé ccl sur l'avantage
Dans cette exemple vous pouvez tester...blablabla
pq cette exemple est intéressant
avantage = plugin, pour maitriser le code , faire du design très spécfique tout en ayant peu de javascript

vraiment la seul bibliothèque qui fait ça 
très précise peu de jS et simple conceptuellement 


en CCl parler de l'ajax
en ccl parler des améliorations possible de coopgui ?

## CSS

## JavaScript
















## License

Coopgui is released under the [MIT license](https://github.com/coopattitude/coopgui/blob/master/LICENSE).





