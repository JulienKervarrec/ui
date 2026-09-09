# Chapitre 3 -- Le navigateur de snapshots : signature SigV4 et reduction de manifeste

La section `app/snapshots/` expose un navigateur de snapshots Reth v2 pour
trois reseaux Base (`mainnet`, `sepolia`, `zeronet`), chacun associe a un
bucket Cloudflare R2 distinct dans le tableau `NETWORKS` de `r2.ts`. Le champ
`hiddenFromUi` sur `zeronet` illustre une distinction volontaire entre deux
notions : un reseau peut rester servi par l API (`/api/snapshots`) parce que
des noeuds continuent de synchroniser depuis son bucket, tout en etant absent
de la page publique -- la fonction `isNetworkVisibleInUi` applique ce filtre
uniquement a la limite du rendu, jamais dans la couche de donnees, pour ne
jamais risquer de couper silencieusement un reseau que des noeuds consomment
encore.

`r2.ts` implemente sa propre signature de requetes S3-compatibles SigV4 a la
main (`signR2Request`), sans dependance au SDK AWS : construction de la
`canonicalRequest` (methode, chemin encode, query string canonique, en-tetes
canoniques, hash du payload vide puisque ce sont des GET), derivation de la
cle de signature par une chaine de HMAC-SHA256 imbriques
(`getSignatureKey` -- date, region, service, puis `aws4_request`), et enfin
calcul de la signature elle-meme. `rfc3986Encode` corrige l encodage par
defaut de `encodeURIComponent`, qui ne pourcent-encode pas certains caracteres
(`! ' ( ) *`) que la norme AWS exige d encoder.

La recherche du snapshot le plus recent (`getLatestSnapshotManifest`) liste
les prefixes du bucket via l API S3 List Objects V2 (avec pagination par
`continuation-token`), filtre pour ne garder que les prefixes purement
numeriques (les hauteurs de bloc), les trie par ordre decroissant, puis
parcourt cette liste du plus recent au plus ancien jusqu a trouver un
`manifest.json` valide -- le commentaire du code precise que le dossier le
plus recent peut encore etre en cours d upload, donc son manifeste peut
temporairement ne pas exister (404), auquel cas le code passe simplement au
suivant.

Le resultat est mis en cache via `unstable_cache` de Next (`SNAPSHOT_CACHE_SECONDS`
= 300s), partage entre la page et la route d API `/api/snapshots`, avec un
choix de fiabilite explicite : si une requete R2 echoue pour un reseau,
`SnapshotLoadError` est levee plutot que de renvoyer une liste partielle qui
serait interpretee comme un succes complet -- et comme Next sert la valeur en
cache perimee quand la revalidation echoue, une liste complete deja en cache
continue d etre servie pendant une panne R2, ce qui borne la fraicheur des
donnees sans masquer l erreur pour autant.

`data.ts` (sans dependance React, partage entre le serveur et le client)
definit les trois presets de telechargement (`archive`, `full`, `minimal`),
chacun avec sa propre liste de composants inclus et de capacites annoncees
(Sync, Validate, Query, Trace, Debug, Index), et la fonction `presetSize` qui
calcule la taille totale d un preset -- avec un cas particulier pour le preset
`full`, qui compte la taille reduite `fullSize` (une fenetre d historique
limitee) plutot que la taille complete `size` pour les composants
historiques comme `transactions` ou `receipts`.
