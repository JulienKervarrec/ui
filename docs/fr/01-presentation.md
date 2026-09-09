# Chapitre 1 -- Presentation de base/ui

Ce depot est le "Base Chain Dashboard" : une seule application Next.js (App
Router) qui reunit plusieurs outils operationnels de Base sous une meme
interface -- le navigateur de snapshots Reth v2, et Vibenet (l explorateur et
le faucet d un devnet interne). Le README precise que ce depot a ete migre
hors d un template Nx interne pour utiliser la chaine d outils Next standard,
et qu il est deploye sur Vercel.

Le point structurant du projet est qu un seul code source produit deux
deploiements distincts, "external" et "internal" (chapitre 2) : le site public
n expose pas certaines sections (comme l Internal Explorer ou le Benchmark),
tandis que le deploiement interne les inclut. Cette dualite traverse toute
l architecture -- navigation, middleware, generation de documentation -- au
lieu d etre un simple flag isole.

Le dossier `app/` contient les routes App Router et leurs gestionnaires d API.
Les sections principales sont `app/snapshots/` (le navigateur de snapshots,
lisant Cloudflare R2 -- chapitre 3), `app/vibenet/` (l explorateur et le
faucet du devnet -- chapitre 4), et `app/components/` (le shell et les
primitives d UI partagees). Les fichiers `theme.ts` et `spectrum.ts`
implementent les tokens de design du Base Design System (BDS), et `public/`
heberge les polices maison (Google Sans Flex, Doto) ainsi que les images.
