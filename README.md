# Coopgui

Coopgui is a php graphics library which allows you to build the front end part of your website very precisely by generating HTML, CSS and JavaScript. __(?)est-ce qu'il faut dire que coopgui génère l'HTML à chaque fois qu'une requete est envoyé (/?)__ 
FB : évident
Coopgui reduces the use of Javascript for security reasons. __(?)oui mais comment elle réduit ? en utilisant le JS que pour la partie graphique ? (=boutons et le responsive design ?) (/?)__

FB:
minimise le js dire que c une des raisons
ça réduit le js car c le php qui génère les éléments html et non le JS. Ce qui est le cas dans plein d'autres... plein d'autes quoi ? des bibliothèque ? 
JS = côté obscure car moins sécurité et d'autres truc (maintenance ?)
:FB

## Prerequisites
__(?)a server with php7 ? (/?)__ 
FB: compatible avec plein de version donc enlever le 7

## Basic Installation
* Copy everything on your server. __(?)Faut-il préciser que pour générer le site il faut exécuter index.php ?(/?)__

FB:
mini tuto d'installation
vider répertoire gen

* Make sure all requests are directed to the [index.php](https://github.com/coopattitude/coopgui/blob/master/index.php) file by your web server configuration.

## coopgui arborescence 
this library works like this: to build a website the user creates daughter classes in [site](https://github.com/coopattitude/coopgui/tree/master/site) and [plugins](https://github.com/coopattitude/coopgui/tree/master/plugins). This classes inherit from the classes inside the system folder. __(à reformuler)__ vérifier classe fils/fille

system : the classes that the user can reuse (=create a daughter class)
plugins:
gen:
js:

FB:
réécrire
pas de détail
le site appel desplugins qui appelle d'autre pollugins et ça permet de monter uns site web
nested plugin explication conceptuellement (détaille plus bas ds #nestedplugin)

## How coopgui works

### HTML generation overview
Coopgui generates the HTML of a web page by calling some of the plugins located in the [plugins](https://github.com/coopattitude/coopgui/tree/master/plugins) directory. When a web browser sends a request, the [site/index.php](https://github.com/coopattitude/coopgui/blob/master/site/index.php) file, which is the main file of the site, is executed. This file successively calls all the plugins it needs and pass them options. Each plugin returns a string containing HTML to the index.php file which then appends everything into an array. Eventually the array is sent using echo. __(dernière phrase à reformuler)__  



## site/index.php
The [site/index.php](https://github.com/coopattitude/coopgui/blob/master/site/index.php) file is the main file of the site.
It defines the class `IrIndex` and immediately creates an IrIndex object (and then calls his init method). __(?)Parler du constructeur et de envcustom, gencss etc ?(/?)__

FB:
paragraphe complet
system de cache
personnalisation de l'environnement
personnalisation du cache

This IrIndex object echos the HTML of the web page. The head method generates the head and the init method generates the body. 

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

__(?)mettre une remarque sur l'ordre d'ajout dans display ? Vu que cela peut modifier la façon dont les éléments s'affichent(/?)__
FB: c évidant

## Plugins
For coopgui to generate a web page you have to write plugins. A plugin represents a part of the page. For example a clock could be a plugin. The plugins must be inside the [plugins](https://github.com/coopattitude/coopgui/tree/master/plugins) directory and contain a PHP file for the HTML generation, a PHP file for the CSS generation and a JavaScript file that contains the code related to the plugin. __(?)encore une fois : est-ce que ce doit être uniquement du JS pour gérer le graphique de manière à être moins attaquable ? Si oui faut-il préciser à l'utilisateur de ne mettre que du JS graphique ?(/?)__
FB: si il veut rajouter du JS non graphique libre à lui

#### How to create a Plugin
Create a folder with the plugin name in the [plugins](https://github.com/coopattitude/coopgui/tree/master/plugins) directory.
##### HTML generation
The plugin's HTML is generated by a PHP class. Below is how to create one. 
* Inside the plugin folder, create a php file with the plugin name. For example [IrHtmlInfo.php](https://github.com/coopattitude/coopgui/blob/master/plugins/Info/IrHtmlInfo.php) for the info plugin.
* Inside this file, define a php class with the same name as the file (the file and the class must have the same name for the autoloader/classloader to work correctly). This class must inherit from the IrHtmlFather class which is located in the [system/Html](https://github.com/coopattitude/coopgui/tree/master/system/Html) directory.
* Add the class to the classloader [IrGuiClassLoader.php](https://github.com/coopattitude/coopgui/blob/master/system/ClassLoader/IrGUIClassLoader.php) as explained in the classloader section of this documentation.
* __(?)redefine the constructor ? (/?)__
FB: évidant
* Create a method `toHtml`. This is the method which will return the HTML of the plugin.
* Inside this method, create an array called "$htmlDef". __(?)Expliquer le unset ?(/?)__

FB:
pas évident ça vide le tableau prend la def php de unset
expliquer plus en commentaire peut être
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
Sometimes you may want to give some data only this class has to the JavaScript. For example the prefixUniq of the plugin. The only way ? is to call a function 

FB:
expliquer le préfixUniq ici
et à quoi il sert et qu'il est unique

(__A finir__)


# nested plugins
détailler fonctionnement

##### CSS generation
A plugin's CSS style sheet is generated by a PHP class. Below is how to create one.
This class have to generate CSS that applies on the tags definied above in the HTML generation class __(?)Est-ce vrai ?(/?)__ 
FB: Oui

parler du prefix css ?
FB:
pas obliger

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
Unlike the HTML generation file, this class doesn't need to be added to the autoloader functions/Classes __(?)(which one should I choose ?)(/?)__ because it is included manually.

FB:
classe = mieux

###### style sheet generation
There is one CSS style sheet for all the plugins. When index.php receives a request, the IrLibGenCss class collects the CSS classes of all the plugins, use them to generate CSS and appends everything into the file css1_all.css in gen/css

FB:
parler du cache ? vérifier que j'en est déjà parler

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


# prefixUniq

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





