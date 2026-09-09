# Chapitre 6 -- Limites et perimetre de ce parcours

Ce parcours couvre l architecture generale de l application (README,
structure `app/`), la matrice de deploiement (`deploy.config.mjs`,
`proxy.ts`), le navigateur de snapshots dans son integralite fonctionnelle
(`r2.ts`, `data.ts`), une partie de la section Vibenet (le faucet et le
catalogue de demos), et le mecanisme de generation de documentation pour
agents ainsi que la CI.

Sont volontairement laisses hors champ, faute de temps plutot que d interet :
le detail des demos individuelles sous `app/vibenet/demos/` (comptes,
tokens B20, et leurs nombreux composants partages dans `_shared/` et
`_components/`) au-dela du catalogue qui les reference ; l explorateur
Vibenet lui-meme (`app/vibenet/explorer` et les composants associes comme
`ExplorerSearch` ou `ExplorerLink`) ; les sections internes uniquement
(`internal-explorer`, `benchmark`) qui ne sont de toute facon pas
accessibles dans le build public etudie ici ; le detail du systeme de design
BDS (`theme.ts`, `spectrum.ts`, les composants `app/components/ui/`) au-dela
de leur simple mention ; et la suite de tests (vitest, Playwright) elle-meme.

L objectif reste le meme que pour les parcours precedents de cette
bibliotheque : comprendre precisement comment ce tableau de bord Base
articule un seul code source en deux deploiements distincts et sert des
donnees operationnelles reelles (snapshots R2, statut du faucet), sans
pretendre couvrir l integralite d une application Next.js de cette taille.
