# Chapitre 5 -- L index pour agents (llms.txt / AGENTS.md) et la CI

Le depot maintient trois fichiers generes automatiquement a partir de l arbre
de routes de l application : `public/llms.txt`, `public/llms-full.txt`, et
`public/AGENTS.md`. Ce sont des index destines a des agents ou modeles de
langage qui explorent le site, produits par les scripts du dossier `scripts/`
(`npm run llms` et `npm run agents`).

Le mecanisme de fraicheur choisi est un hook git `post-commit` local plutot
qu une etape CI qui bloquerait la fusion : des qu un commit mentionne
`llms.txt` ou `agents.md` dans son message, ou ajoute/renomme un fichier de
route, le hook regenere ces fichiers dans un commit de suivi automatique.
Le hook est desactive par defaut a chaque nouveau clone -- `core.hooksPath`
est un reglage git local qui ne peut pas etre commite dans le depot lui-meme,
donc chaque personne qui clone doit executer `./githooks/install.sh` pour l
activer explicitement (le script precise qu il ne touche jamais a la
configuration git globale ou systeme). Un commit individuel peut contourner
le hook via `SKIP_DOCS_HOOK=1 git commit` ou un message contenant
`[skip-docs]`.

La CI (`.github/workflows/ci.yml`, execute sur chaque pull request et sur les
push vers la branche par defaut) combine plusieurs verifications
independantes : `typecheck` (`tsc --noEmit`), `lint` (eslint), `test`
(vitest -- avec une mention specifique du test de contrat
`networks.contract.test.ts`, qui verifie que chaque reseau attendu reste
servi par `/api/snapshots`, empechant qu un reseau soit retire de l API sous
pretexte qu il a ete cache de l UI via `hiddenFromUi`), une etape `docs` qui
echoue si les fichiers generes sont perimes par rapport a l arbre de routes
reel, une etape qui construit specifiquement la cible externe et verifie que
les routes internes renvoient bien 404 et sont absentes de la navigation et
du sitemap (empechant une regression silencieuse de la matrice de deploiement
du chapitre 2), et enfin une suite de tests de bout en bout Playwright
marquee `continue-on-error` -- volontairement non bloquante, le temps que
cette suite se stabilise, mais dont les echecs restent visibles dans les
resultats de CI.
