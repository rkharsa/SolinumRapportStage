<h1>Rapport de Stage</h1>
<h2>Soliguide c'est quoi ?</h2>
Soliguide,est  le guide de la solidarité numérique pour personnes sans-abri ou réfugiés. Se doucher, manger, domiciliation : toutes les informations utiles sont referencées dedans ! il permet d'avoir acces à differentes informations à jour  sur les structures rapidement. 
<h2>Cas d'utilisation</h2>
<a href="http://zupimages.net/viewer.php?id=19/14/h77a.png"><img src="https://zupimages.net/up/19/14/h77a.png" alt="" /></a>
<h2>Model de données</h2>
<a href="http://zupimages.net/viewer.php?id=19/14/cjnc.png"><img src="https://zupimages.net/up/19/14/cjnc.png" alt="" /></a>
<h2>Amelioration du moteur de recherche</h2>
Après de nombreux retours nous avons amélioré le moteur de recherche .
En effet le moteur de recherche ne permettait pas de retrouver des structures à partir d'un département, région ou ville mais qu'à partir de l'adresse car dans la base de données ces données n'étaient pas mémorisées.
Pour cela nous avons dû mettre à jour la structure de la base données pour lui permettre de mémoriser ces données nous sommes en NoSql pour modifier la structure de la base donnée il faut :
<ul>
<li>Modifier le schéma de base de données (le modèle) qui se trouve dans l'api (dans le backend)</li>
<li> Modifier le schema de base de données (le model) dans le front au niveau de l'insertion des données donc principalement cote admin du front  </li>
</ul>
Après avoir modifié la structure de la base donnée il faut l'alimenter et mettre à jour les anciens Documents(en NoSql on parle principalement de document c'est l'équivalent en SQL d'une entree).
Avant de vous expliquer comment j'ai fait pour l'alimenter je vais vous résumer le fonctionnement de l'api de google qui joue un rôle-clé dans la solution. l'API de google map permet à partir d'une adresse, d'un pays, département, région de récupérer des données les concernant .
L'API de google map on peut la représenter comme une échelle en effet en fonction du type de données envoyées (pays, département, région, adresse) on a different schéma de données envoyer.
exemple de fichier json récupéré après une requête http
<pre><code>
{
   "results" : [
      {
         "address_components" : [
            {
               "long_name" : "6",
               "short_name" : "6",
               "types" : [ "street_number" ]
            },
            {
               "long_name" : "Rue Claude Farrère",
               "short_name" : "Rue Claude Farrère",
               "types" : [ "route" ]
            },
            {
               "long_name" : "Paris",
               "short_name" : "Paris",
               "types" : [ "locality", "political" ]
            },
            {
               "long_name" : "Arrondissement de Paris",
               "short_name" : "Arrondissement de Paris",
               "types" : [ "administrative_area_level_2", "political" ]
            },
            {
               "long_name" : "Île-de-France",
               "short_name" : "Île-de-France",
               "types" : [ "administrative_area_level_1", "political" ]
            },
            {
               "long_name" : "France",
               "short_name" : "FR",
               "types" : [ "country", "political" ]
            },
            {
               "long_name" : "75016",
               "short_name" : "75016",
               "types" : [ "postal_code" ]
            }
         ],
         "formatted_address" : "6 Rue Claude Farrère, 75016 Paris, France",
         "geometry" : {
            "location" : {
               "lat" : 48.8426547,
               "lng" : 2.2533314
            },
            "location_type" : "ROOFTOP",
            "viewport" : {
               "northeast" : {
                  "lat" : 48.8440036802915,
                  "lng" : 2.254680380291502
               },
               "southwest" : {
                  "lat" : 48.8413057197085,
                  "lng" : 2.251982419708498
               }
            }
         },
         "place_id" : "ChIJSWB-rMB65kcR4PcGLvJ9MR8",
         "plus_code" : {
            "compound_code" : "R7V3+38 Paris, France",
            "global_code" : "8FW4R7V3+38"
         },
         "types" : [ "street_address" ]
      }
   ],
   "status" : "OK"
}
</code></pre>
<ul>
   <li>5-Pays</li>
   <li>4-Region</li>   
   <li>3-Departement</li>   
   <li>2-Ville</li>
   <li>1-Adresse</li>
</ul>
À partir d'une adresse on accède à la ville,au département, région, pays alors qui si on recherche à partir de département on aura accès qu'ont la région et pays .
Pour mettre à jour les données nous devons creer une fonction dans le côté backend de la solution qui va se dérouler ainsi ;
<ul>
  <li>Requete MongoDb on va rechercher tous les documents qui n'ont pas de ville et logiquement par département ....</li>
   <pre><code>
   Lieu.findOne({ ville: null }).lean().exec(function (err, lieu) {...}
   </code></pre>
   <li>Ensuite pour chaque document trouvé on fait une requête http de l'API de google et rechercher en fonction de l'adresse</li>
      <pre><code>
   axios.get("https://maps.googleapis.com/maps/api/geocode/json?address=" + encodeURIComponent(lieu.address) + "&key=AIzaSyDMsYajtLOcjtima174SSmshCA3VPtRxVk").then(response => {}
   </code></pre>
   <li>on verifie les données renvoyer a l'aide de conditions</li>
     <pre><code>
   axios.get("https://maps.googleapis.com/maps/api/geocode/json?address=" + encodeURIComponent(lieu.address) + "&key=AIzaSyDMsYajtLOcjtima174SSmshCA3VPtRxVk").then(response => {
    data = response.data;    
        for (var x = 0; x < data.results[0].address_components.length; x++) {
          for (var y = 0; y < data.results[0].address_components[y].types.length; y++) {   
            if (data.results[0].address_components[x].types[y] == "administrative_area_level_1") {
              regionNew = data.results[0].address_components[x].long_name;
            }
            if (data.results[0].address_components[x].types[y] === "country") {
              paysNew = data.results[0].address_components[x].long_name;
            }
            if (data.results[0].address_components[x].types[y] === "postal_code") {
              codePostalNew = data.results[0].address_components[x].long_name;
            }
            if (data.results[0].address_components[x].types[y] === "administrative_area_level_2") {
              departementNew = data.results[0].address_components[x].long_name;
            }
            if (data.results[0].address_components[x].types[y] === "locality") {
              villeNew = data.results[0].address_components[x].long_name;
            }
   }
   </code></pre>
   <li>on met a jour  dans la bdd</li>
    <pre><code>
   Lieu.updateOne({ lieu_id: lieu.lieu_id }, {
          $set: {
            ville: villeNew,
            departement: departementNew,
            codePostal: codePostalNew,
            pays: paysNew,
            seo_url: newSeoUrl,
            region: regionNew
          }
        }).exec(function (err, retour) {
          if (err) {
            console.log(err);
            return callback(err, null);
          }
          if (retour.ok === 1) {
            self.updateLocalisation();
          }
        });
   </code></pre>
  </ul>
   <h2>Amelioration de l'ergonomie  de saisie des horaires</h2>
   <a href="http://zupimages.net/viewer.php?id=19/13/q6jq.png"><img src="https://zupimages.net/up/19/13/q6jq.png" alt="" /></a>
   Algorithmie
   <ul>
   <li>On vérifie si le jour passe fermé ou ouvert( en effet cette fonction se génère lors d'un événement de changement lorsqu'on coche et decoche la case jour</li>
   <li> S'il passe en fermer alors efface les données contenues dans le champ</li>
   <li>Si il passe en ouvert 
      <ul>
   <li>On trouve le dernier jour ouvert</li>
   <li>On récupère les horaires du dernier jour ouvert</li>
   <li>On les copies dans le nouveau jour ouvert</li>
      </ul></li>
   <li> Si la structure n'a pas d'horaire au jour d'avant  on met des horaires par default 9h -17h</li>
  
</ul>
 <pre><code>
 $scope.changeDay = function (hours, jour) {
    if (hours[jour].open === false)
    {
      hours[jour].openAfternoon = false;
      $scope.deleteHours(hours[jour], true);
      $scope.deleteHours(hours[jour], false);
    } 
    else {
      var dernierJour = $scope.dernierJourOuvert(hours, jour);
      if (jour === dernierJour) {
        var dateMatin = new Date();
        var dateAprem = new Date();
        dateMatin.setHours(9, 0, 0, 0);
        dateAprem.setHours(17, 0, 0, 0);   
        hours[jour] = {
          evening_start: dateMatin,
          evening_end: dateAprem,
          afternoon_start: '',
          afternoon_end: '',
          open: true,
          openAfternoon: false
        };
      }
      else {
        hours[jour] = angular.copy(hours[dernierJour]);  
      } 
    }
  };
   </code></pre>
   <h2>Mise en place de compteur dynamique</h2>
   <a href="http://zupimages.net/viewer.php?id=19/13/epg5.png"><img src="https://zupimages.net/up/19/13/epg5.png" alt="" /></a>
   <h3>Backend</h3>
   Mise en place de la fonction et de la requete mongoDb qui permet compter le nombre de Lieux referencer et services referencer
   <h4>Compteur de lieux</h4>
   <pre><code>
   this.countAll = function (query, callback) {
  var opt = this.createQuery(query);
  nosql_query = opt.nosql_query;
  options = opt.options;
  Lieu.count(nosql_query).exec(function (err, nbResults) {
    if (err) {
      return callback(err.msg, null);
    }
    callback(null, nbResults);
  });
};
</pre></code>
<h4>Compteur  de service</h4>
<pre><code>
this.countAllService = function (query, callback) {
  Lieu.aggregate([
    {
      "$project": {
        "totalService": { "$size": "$services_all" }
      }
    },
    {
      "$group": {
        "_id": null,
        "count": {
          "$sum": "$totalService"
        }
      }
    }]).exec(function (err, nbResults) {
      callback(null, nbResults);
    });
    
  };
  </code></pre>
  <h3>FrontEnd</h3>
  <h4>Compteur de lieux</h4>
  <pre><code>
  	$http({
		method: 'GET',
		headers: $window.headers,
		url: $window.host + 'lieux/countAll/'
	}).then(function (nombre) {
		$scope.totalLieu = nombre.data;
	});
</code></pre>
<h4>Compteur de services</h4>
  <pre><code>
	$http({
		method: 'GET',
		headers: $window.headers,
		url: $window.host + 'lieux/countAllService'
	}).then(function (nombre) {
		$scope.totalService = nombre.data[0].count;
	});
	
  </code></pre>
   
   <h2>Ajouter un 2 eme numero de telephone</h2>
   <h3>Backend</h3>
   <ul><li>Ajout dans la structure de données  le model lieu</li></ul>
   <h3>FrontEnd</h3>
   <ul><li>Ajouter le champ</li></ul>
   <a href="http://zupimages.net/viewer.php?id=19/13/x7h2.png"><img src="https://zupimages.net/up/19/13/x7h2.png" alt="" /></a>
   <h2>Import de CSV </h2>
   Cette fonction permet d'importer des fichiers csv dans la base de données
   Etape :
   <ul>
	<li>Lire le fichier CSV</li>
	<li>Recuperer les données</li>
	<li>Faire correspondre au model de données</li>
	<li>Inserer le document dans la bdd</li>
	<li>Recommencer jusqu'a la fin de la lecture du fichier csv (Fonction recursive)</li>
   </ul>
   <pre><code>
     this.importDataCSV = function (query, callback) {
    var fileName = 'montpellier.json'; 
    var path = global.ENV === 'dev' ? "./public/" + fileName : "/root/api_solinum/public/" + fileName
    fs.readFile(path, function read(err, params) {
      if (err) {
        console.log(err);
        throw err;
      }
      var tableau;
      var structureModel = {};
      var test = JSON.parse(params);
      tab = test["Sheet 1"];
      //tab = test["features"];
      var i = 0;
      console.log("LANCEMENT");
      self.addLieuCsv(i, tab);
    });
  };
   </code></pre>

   <h2>Automatisation des structures fermer temporairement</h2>
<h3>Backend</h3>
<ul>
	<li>Ajout au model de données de l'objet Close</li>
	<pre><code>  
	close: {
    closeType: {
      type: Number,
      default: 0 /* 0 ouvert / 1 définitivement / 2 temporairement */
    },
    precision: {
      type: String,
      default: null,
    },
    dateDebut: {
      type: Date,
      default: null
    },
    dateFin: {
      type: Date,
      default: null
    }
  }</pre></code>
	<li>Mise a jour du model dans mongoDb  et des anciens documents enregistrer(Avec des requetes mongo cf.fait par mon tuteur)</li>
</ul>
<h3>Front End</h3>
<ul>
	<li>Mise a jour du model de donnée dans le controller Admin</li>
	<li>Ajout d'une div qui s'affiche a une condition si la date  actuelle se trouve entre la date de debut et de fin du fermer temporairement.C'est possible grace a  "ng-if" avec angular qui affiche en fonction de la condition.</li>
</ul>

