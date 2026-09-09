# Chapitre 2 -- La matrice de deploiement : deploy.config.mjs et le middleware

`deploy.config.mjs` est le fichier central qui declare, pour chaque "surface"
(section de l application) non universelle, dans quelles cibles de build elle
doit apparaitre. La cible active, `TARGET`, est lue depuis la variable d
environnement `NEXT_PUBLIC_DEPLOY_TARGET` au moment du build (`external` par
defaut, `internal` si positionnee) -- c est donc une valeur figee a la
compilation, pas une bascule runtime.

La table `SURFACES` associe a chaque section optionnelle ses prefixes de
routes UI (`routePrefixes`), ses prefixes d API (`apiPrefixes`), et la liste
des cibles ou elle doit etre presente (`targets`). Le depot en declare deux
aujourd hui : `internal-explorer` (avec `/tips` et `/api/tips` conserves comme
anciens prefixes, pour que le build public renvoie un 404 explicite sur ces
URLs historiques plutot que de les rediriger) et `benchmark` (qui n a pas d
`apiPrefixes` car son UI appelle directement une API externe via
`NEXT_PUBLIC_BENCHMARK_API_BASE_URL`, sans passer par une route de ce depot).
La fonction `surfaceEnabled(key)` fait le test inverse : une cle absente de
`SURFACES` est consideree activee partout, ce qui evite d avoir a lister
explicitement toutes les sections universelles.

Le fichier expose ensuite des fonctions derivees consommees par differentes
couches : `disabledRoutePrefixes()` pour le middleware, `disabledApiPrefixes()`
pour les gardes d API individuelles (documentation, pas application directe),
et `disabledRouteGlobs()` pour le generateur de fichiers `llms.txt`. Le
commentaire du fichier insiste sur un point de securite : la garantie offerte
est l inaccessibilite (les routes renvoient 404), pas l absence de bundle --
le code client d une section desactivee peut toujours etre present dans le
JavaScript livre, ce qui est juge acceptable puisque le depot est public.

`proxy.ts`, le middleware Next.js, applique cette regle a la frontiere de
chaque requete : pour tout chemin qui commence par un prefixe present dans
`disabledRoutePrefixes()`, il renvoie directement un 404 avant meme d
atteindre la page. Le commentaire du fichier explique pourquoi ce blocage cote
middleware est indispensable et pas seulement decoratif : une page desactivee
peut avoir ete pre-rendue statiquement, auquel cas son `notFound()` produit un
contenu de type 404 mais avec un statut HTTP 200 -- c est le middleware qui
impose le vrai code 404. Le `matcher` du middleware exclut explicitement
`/api/`, les fichiers internes Next (`_next/static`, `_next/image`), et
quelques fichiers statiques racine, pour ne pas intercepter ce qui n a pas
besoin de l etre.
