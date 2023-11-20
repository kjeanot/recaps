# Les tests côté back-end à la soumission d'un formulaire

## Tests de valeurs soumises  

1- On récupère les données du formulaire et on applique un filter_input pour valider le type du champ à récupérer.

```PHP

        $name = filter_input(INPUT_POST, 'name', FILTER_SANITIZE_STRING);   // en PHP 8.1 => on utiliser htmlspecialchars()
        $subtitle = filter_input(INPUT_POST, 'subtitle', FILTER_SANITIZE_STRING);
        $picture = filter_input(INPUT_POST, 'picture', FILTER_VALIDATE_URL);

        // Pour récupérer un INT on utilise le filtre FILTER_VALIDATE_INT

```

2- (Facultatif) : on peut vider les espaces (au début et à la fin) en utilisant trim()

```PHP
        // Si on souhaite, on pourrait vider les espaces, en utilisant trim()
        // Ca enlève que au début et à la fin les espaces

        // Si la chaine est remplie, j'effectue le nettoyage, sinon je laisse la chaine telle quelle
        if(!empty($name)) {
            $name = trim($name);
        }
        $subtitle = (!empty($subtitle)) ? trim($subtitle):$subtitle;
        $picture = (!empty($picture)) ? trim($picture):$picture;

```

3- Vérif côté back en php

On vérifie si le champ est vide ou si le filter_input renvoit false. Si une condition d'erreur est rencontrée, dans ce cas on stocke un message d'erreur dans un tableau.

```PHP
// 2- Vérifications (côté back end)

        // Vérifications sur le champs NAME
        if(empty($name)) {      // si c'est vide ?
            $errorList[] = 'Le nom est vide';
        }
        if($name === false) {   // est-ce que la valeur du champs name a bien réussi à passer le filtre
            $errorList[] = 'Le nom est invalide';
        }
        // Vérifications sur le champs SOUS TITRE
        if(empty($subtitle)) {      // si c'est vide ?
            $errorList[] = 'Le sous-titre est vide';
        }
        if($subtitle === false) {   // est-ce que la valeur du champs a bien réussi à passer le filtre
            $errorList[] = 'Le sous-titre est invalide';
        }
        // Vérifications sur le champs IMAGE
        // Comme l'image n'est pas obligatoire dans la base, on a pas besoin de tester le empty
        if($picture === false) {   // est-ce que la valeur du champs a bien réussi à passer le filtre
            $errorList[] = 'l\'URL d\'image est invalide';
        }

```

Si le tableau est vide une fois les tests passés, on peut exécuter notre requête SQL en :

- Créant une nouvelle instance de notre model concerné
- Renseignant les attributs de ce nouvel objet à l'aide des setters
- Exécutant notre méthode insert, si elle ne renvoit pas d'erreur

```PHP
// 3- On s'assure qu'il n'y a aucune erreur (grâce à des variables)
        // => Est-ce que le tableau errorList est vide ?
        if(empty($errorList)) {
            // 3a- On fait la requete SQL INSERT
            // ici je suis sûr d'avoir protéger correctement ma BDD, et je peux maintenant faire la requete
            $category = new Category();
            $category->setName($name);
            $category->setSubtitle($subtitle);
            $category->setPicture($picture);

            if($category->insert()) {
                // 3b- On redirige l'utilisateur vers la liste des catégories
                // Attention, de ne pas laisser trainer de var_dump dans le code !!!

                // Pourquoi je n'utilise pas la méthode show() => 
                // PARCE QUE SI JE FAIS F5, je vais renvoyer à l'infini les données du formulaire !
                // DONC => créer autant de lignes que de F5 que je vais faire sur la page
                // DONC CREATION DE DOUBLONS EN BDD

                header('Location: /categories/list');
                exit;   // par sécurité, pour pas que la suite se déroule
            } else {
                $errorList[] = "La sauvegarde a échouée";
            }
        }

```

Si les tests relèvent des erreurs, elles seront listées dans le tableau. Ce tableau est stocké dans une variable qu'on va pouvoir passer dans la vue pour afficher les erreurs.
on affiche dans chaque input en erreur la valeur que l'utilisateur a saisi (ou pas). Pour ce faire, on crée une nouvelle instance du Model concerné par l'ajout des données, on définit ses attributs, de façon à pouvoir renvoyer un objet dans la vue.

```PHP

        // 4- Sinon, dans le cas contraire, afficher le formulaire avec les erreurs
        if(!empty($errorList)) {

            //var_dump($errorList);

            // Plutôt que de passer les valeurs à notre formulaire directement
            // on va instancier un objet de type Category
            // Attention, nous avons un objet, mais qui n'est pas persisté, puisqu'on affiche ici les erreurs
            $category = new Category();

            // Ici attention à renvoyer les données brutes
            // comme ça, l'utilisateur récupère ses valeurs d'origine (tapées)
            $category->setName(filter_input(INPUT_POST, 'name'));
            $category->setSubtitle(filter_input(INPUT_POST, 'subtitle'));
            $category->setPicture(filter_input(INPUT_POST, 'picture'));

            $this->show('catalog/category-add', [
                'category' => $category,        // les données qu'on a soumis la première fois !
                'errorList' => $errorList
            ]);
        }

```

Comme on a indiqué un tableau contenant notre liste d'erreurs en argument de la fonction show, on peut utiliser la variable $errorList dans notre vue et afficher les messages qu'elle contient : 

```PHP
<?php foreach($errorList as $error): ?>
        <div class="alert alert-danger" role="alert">
            <strong>Erreur :</strong> <?= $error ?>
        </div>
<?php endforeach; ?>
```

On peut aussi afficher la valeur des champs n'ayant pas passé nos tests en back-end pour guider l'utilisateur :

```PHP
        <div class="mb-3">
            <label for="name" class="form-label">Nom</label>
            <input type="text" class="form-control" id="name" name="name" placeholder="Nom de la catégorie"
                value="<?= (!empty($errorList)) ? $category->getName() : '' ?>"
            >
        </div>
```

## Envoi en base de données

à détailler.
