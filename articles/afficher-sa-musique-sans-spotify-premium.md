---
title: Afficher sa musique sur son site quand Spotify dit non
summary: Depuis 2026, l'API Spotify exige un compte Premium. Je l'ai remplacée par deux sources gratuites, Last.fm pour l'historique et Lanyard pour le direct, et j'y ai plutôt gagné.
tags: [api, lastfm, lanyard, dotnet]
date: 2026-11-01
---

Je voulais une chose simple sur mon portfolio : montrer ce que j'écoute pendant que je code. Le morceau en cours, ce qui tourne en boucle, à quelles heures. Le plan évident, c'était l'API Spotify. Il a tenu une soirée.

## Le mur

En février 2026, Spotify a [durci son mode développement](https://techcrunch.com/2026/02/06/spotify-changes-developer-mode-api-to-require-premium-accounts-limits-test-users/). Depuis le 11 février pour les nouvelles applications, et depuis le 9 mars pour toutes les autres, le propriétaire de l'app doit avoir un abonnement Premium, et l'app est limitée à cinq utilisateurs de test. Spotify dit vouloir freiner les usages automatisés.

Mon compte est gratuit. Fin de l'histoire côté Spotify.

:::avatar{mood=doubt}
Payer un abonnement chaque mois pour afficher quatre lignes de texte sur un site perso ? Non.
:::

En listant ce dont j'avais vraiment besoin, il restait quatre choses : le titre en cours, sa position (pour une barre de progression), un historique avec des classements, et pas d'abonnement. Aucun service gratuit ne fait tout. Deux le font à eux deux.

Le tableau ci-dessous est branché sur les vraies sources : il montre ce que chacune sait de ma musique, maintenant.

::sources

## Last.fm pour l'historique

Last.fm enregistre chaque écoute, c'est le *scrobbling*. Spotify s'y relie gratuitement, depuis les réglages de Last.fm (Applications, puis Spotify Scrobbling), et une clé d'API s'obtient en deux minutes.

```terminal title="zsh"
# Le dernier titre, en cours ou non
$ curl -s "https://ws.audioscrobbler.com/2.0/?method=user.getrecenttracks&user=$LASTFM_USER&api_key=$LASTFM_KEY&format=json&limit=1"
{"recenttracks":{"track":[{"name":"…","artist":{"#text":"…"},"@attr":{"nowplaying":"true"}, …}]}}
```

Un détail compte : un scrobble n'est pas une écoute. Last.fm ne compte un titre qu'à partir d'un certain point, fixé par [sa règle officielle](https://www.last.fm/api/scrobbling#when-is-a-scrobble-a-scrobble). Le plus simple est de la manipuler :

::scrobble

L'historique a donc toujours un temps de retard, et Last.fm ne connaît pas la position dans le titre. Il signale bien le morceau en cours (l'attribut `nowplaying`), mais sans dire où j'en suis.

### Une API entre les deux

Le site n'appelle jamais Last.fm directement. Une petite API en C# ([api-portfolio](https://github.com/Neptune2k21/api-portfolio), .NET 10 compilé en Native AOT) garde la clé côté serveur et met chaque réponse en cache, avec une durée calée sur la vitesse à laquelle la donnée change : 30 secondes pour le dernier titre, une heure pour les classements.

```csharp title="MusicService.cs"
public Task<Scrobble?> GetLatestAsync(CancellationToken ct) =>
    cache.GetOrLoadAsync("music:latest", TimeSpan.FromSeconds(30),
        async t => (await lastFm.GetRecentAsync(1, t)).FirstOrDefault(), ct);
```

Ce cache fusionne aussi les requêtes simultanées : cent visiteurs dans la même seconde, c'est un seul appel à Last.fm.

Dernier piège : Last.fm ne fournit plus de vraies photos d'artistes. À la place, il renvoie une image générique, toujours la même. L'API la repère à son nom de fichier et va chercher le visuel sur Deezer, dont la recherche est publique et sans clé.

## Lanyard pour le direct

Pour le direct, la solution est passée par Discord. Quand Spotify est relié à Discord, mon profil affiche « Écoute Spotify », avec l'heure de début et de fin du titre. [Lanyard](https://github.com/Phineas/lanyard), un projet open source, rend cette présence lisible par n'importe quel site, en REST ou en WebSocket. Il suffit de rejoindre son serveur Discord.

Les deux sources suivent donc deux chemins différents :

::flow{label="En direct" steps="Spotify, Discord, Lanyard, Ton navigateur" note="WebSocket, poussé à chaque changement" tone=mint}

::flow{label="Sur la durée" steps="Spotify, Last.fm, API C#, Next.js" note="REST, en cache de 30 s à 1 h" tone=sky}

Côté navigateur, le protocole tient en quelques lignes : Lanyard dit bonjour avec un intervalle de battement, on s'abonne à un identifiant Discord, puis chaque changement de présence arrive tout seul.

```ts title="presence.tsx"
ws.onmessage = (e) => {
  const msg = JSON.parse(e.data)
  if (msg.op === 1) {
    // Hello : on s'abonne puis on bat la mesure au rythme demandé
    beat = setInterval(() => ws?.send(JSON.stringify({ op: 3 })), msg.d.heartbeat_interval)
    ws?.send(JSON.stringify({ op: 2, d: { subscribe_to_id: DISCORD_ID } }))
  } else if (msg.op === 0 && (msg.t === "INIT_STATE" || msg.t === "PRESENCE_UPDATE")) {
    setPresence(toPresence(msg.d))
  }
}
```

La position se déduit des deux horodatages : le navigateur la recalcule chaque seconde, sans rien redemander. Discord fournit aussi l'identifiant Spotify du titre, ce qui donne un bouton « écouter avec moi » vers `open.spotify.com/track/…`, et l'URL de la pochette, dont le site tire une couleur pour teinter la carte.

:::avatar{mood=surprised}
Lanyard est public : tout ce que Discord montre de moi, n'importe qui peut le lire. Mon extension VS Code y envoie même le nom du fichier ouvert. Le site n'en garde que le langage.
:::

## Ce que j'ai perdu, ce que j'ai gagné

| | API Spotify | Last.fm + Lanyard |
|---|---|---|
| Coût | Premium | 0 € |
| Titre en cours et position | Oui | Oui, par Lanyard |
| Historique | Les 50 derniers titres | Tout, depuis l'inscription |
| Classements | Oui | Oui, par période |
| Contrôler la lecture | Oui, avec Premium | Non |
| Discord fermé | Rien ne change | Plus de direct, l'historique reste |

Je ne pourrai jamais lancer un titre depuis le site, et le direct disparaît quand Discord est fermé. En échange, j'ai un historique complet au lieu de cinquante titres, et deux sources qui ne dépendent d'aucun abonnement.

Mon historique Last.fm commence le 30 octobre 2026 : les classements « depuis toujours » de la [page Musique](/musique) sont encore jeunes. Ils se rempliront tout seuls. Et la bande-son affichée à côté de cet article sort des mêmes données : c'est ce que j'écoutais en l'écrivant.
