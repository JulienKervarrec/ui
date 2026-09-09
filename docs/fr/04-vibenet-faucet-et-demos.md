# Chapitre 4 -- Vibenet : faucet et catalogue de demos

`app/vibenet/` heberge l explorateur et le faucet d un devnet interne appele
Vibenet, ainsi qu une serie de demos d account abstraction et de tokens B20
regroupees sous `app/vibenet/demos/`.

La page du faucet (`faucet/page.tsx`, un composant client) illustre le style
general des pages Vibenet : elle interroge periodiquement le statut du faucet
via `vibenetApi.faucet.status()` (rafraichi toutes les 15 secondes par
`window.setInterval`), et determine l adresse par defaut a preremplir avec
`defaultFaucetRecipient` (`recipient.ts`) -- une fonction pure et delibrement
minimale qui prefere le compte actuellement selectionne dans les demos
(`activeAccountId`), puis retombe sur le premier compte enregistre, puis sur
`null` si la liste est vide. Une fois que le visiteur tape ou colle une valeur
dans le champ adresse, cette saisie prend le pas sur la valeur par defaut --
y compris un champ intentionnellement vide, un detail explicite dans le
commentaire du composant. La gestion d erreur distingue le cas d un rate
limit HTTP 429 (message specifique "rate limited -- wait a minute and try
again") des autres erreurs de l API, via le type `VibenetApiError`.

Le catalogue des demos (`demos/catalogue.ts`) centralise, dans un tableau de
donnees pur `DEMOS` sans dependance serveur, toutes les entrees affichees sur
la page d accueil Vibenet -- titre, resume, points forts, disponibilite,
icone. Le commentaire du fichier explique pourquoi cette centralisation
compte : la page d index construit ses cartes a partir de ce tableau et le
fil d Ariane (breadcrumb) resout le libelle d une route depuis la meme
source, donc les deux ne peuvent pas diverger. Le champ `listed` (par defaut
`true`) permet de garder une route joignable par URL directe tout en la
retirant de la grille publique -- une demo peut donc etre "live mais non
annoncee", un mecanisme du meme esprit que `hiddenFromUi` pour les reseaux de
snapshots (chapitre 3), applique cette fois au niveau des demos plutot que
des reseaux.
