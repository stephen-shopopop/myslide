---
# try also 'default' to start simple
theme: seriph
# titleTemplate pour la page Web, `%s` sera remplacé par le titre de la page
titleTemplate: '%s - EventEmitter'
# téléchargement de pdf activé dans la version SPA, peut également être une URL personnalisée
download: true
# nom de fichier du fichier d'exportation
exportFilename: 'eventEmitter-exported'
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/298137/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: true
# some information about the slides, markdown enabled
info: |
  ## Team's Shopopop
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
# Fonts
fonts:
  # le texte
  sans: 'Robot'
  # utiliser avec la classe css `font-serif` de windicss
  serif: 'Robot Slab'
  # pour les blocs de code, le code en ligne, etc.
  mono: 'Fira Code'
---

# Welcome to team's Shopopop

Presentation slides for developers

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://www.shopopop.com" target="_blank" alt="Shopopop"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:debug />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
layout: cover
class: text-center
---

# Classe EventEmitter

```ts
const eventEmitter = new EventEmitter();
```

<img src="https://nodejs.dev/static/images/brand/logos-js-right/light.svg" class="m-auto" />

---
layout: center
---

# Architecture événementielle

Une grande partie de l'API principale de Node.js est construite autour d'une architecture événementielle asynchrone idiomatique dans laquelle certains types d'objets (appelés "émetteurs") émettent des événements nommés qui provoquent l'appel d'objets Function ("auditeurs").

Par exemple : un objet net.Server émet un événement chaque fois qu'un pair s'y connecte ; un fs.ReadStream émet un événement lorsque le fichier est ouvert ; un flux émet un événement chaque fois que des données sont disponibles pour être lues.

---
layout: center
---

# Aller plus loin dans les origines

EventEmitter utilise les cycles de boucle d'événements de base de LibUV pour délivrer des événements et exécuter des rappels, ce qui signifie que lorsque vous émettez un événement, il va être ajouté dans la pile de déclenchement d'événements de LibUV pour être déclenché lorsqu'il y a un temps de synchronisation disponible pour cette opération.

---
layout: center
---

# EventEmitter

Tous les objets qui émettent des événements sont des instances de la classe EventEmitter. Ces objets exposent une fonction eventEmitter.on() qui permet d'attacher une ou plusieurs fonctions à des événements nommés émis par l'objet. Généralement, les noms d'événements sont des chaînes en casse camel, mais n'importe quelle clé de propriété JavaScript (ex: Symbol) valide peut être utilisée.

Lorsque l'objet EventEmitter émet un événement, toutes les fonctions attachées à cet événement spécifique sont appelées de manière synchrone. Toutes les valeurs renvoyées par les écouteurs appelés sont ignorées et rejetées.

```ts
import { EventEmitter } from 'node:events';

const myEmitter = new EventEmitter();

myEmitter.on('event', () => {
  console.log('an event occurred!');
});

myEmitter.emit('event');
```

---
layout: center
---

# Passez des arguments

La méthode eventEmitter.emit() permet de transmettre un ensemble arbitraire d'arguments aux fonctions d'écoute. Gardez à l'esprit que lorsqu'une fonction d'écouteur ordinaire est appelée, la norme this mot-clé est intentionnellement définie pour référencer l'instance EventEmitter à laquelle l'écouteur est attaché.

```ts
import { EventEmitter } from 'node:events';

const myEmitter = new EventEmitter();

myEmitter.on('event', function(a, b) {
  console.log(a, b, this, this === myEmitter);
});

myEmitter.emit('event', 'a', 'b');
```

---
layout: center
---

# EventEmitter asynchrone

L'EventEmitter appelle tous les écouteurs de manière synchrone dans l'ordre dans lequel ils ont été enregistrés. Cela garantit le bon séquencement des événements et permet d'éviter les conditions de course et les erreurs logiques. Le cas échéant, les fonctions d'écoute peuvent basculer vers un mode de fonctionnement asynchrone à l'aide de setImmediate() ou process.nextTick()

```ts
import { EventEmitter } from 'node:events';

const myEmitter = new EventEmitter();

myEmitter.on('event', (a, b) => {
  setImmediate(() => {
    console.log('this happens asynchronously');
  });
});

myEmitter.emit('event', 'a', 'b');
```

---
layout: center
---

# Issues EventEmitter

## Les écouteurs.

Le problème d'avoir trop d'écouteurs. Par défaut, EventEmitter veut que nous maintenions le nombre d'écouteurs aussi bas que possible car sur chacun, il exécute une boucle de synchronisation sur les rappels, ce qui bloque toute la boucle d'événements.

La limite initiale est de seulement 25 abonnés par événement, ce qui est tout à fait acceptable pour une application moyenne, MAIS vous pouvez augmenter ce nombre autant que vous le souhaitez. Le principal inconvénient d'avoir de grands nombres est le coût des performances du processeur qui en découle.

## Le problème du maintien du niveau de concurrence.

Lorsque vous faites tourner N fois une opération asynchrone, cela crée une file d'attente de promesses dans le pool de threads, cela signifie que si vous émettez un événement (qui est synchronisé), N fois la file d'attente se développe de la même manière. Pour Node.js, cela pourrait entraîner des plantages de dépassement de mémoire ou d'autres erreurs inattendues.

---
layout: center
---

# Conclusion

EventEmitter n'est pas pour chaque cas d'utilisation d'application, et vous pouvez certainement le remplacer par une implémentation personnalisée, MAIS le plus important est de garder à l'esprit qu'EventEmitter est lié aux événements de LibUV qui est le principal moteur de boucle d'événements pour Node .js.

---
layout: center
---

# En savoir plus ...

- [EventEmitter](https://nodejs.dev/fr/learn/the-nodejs-event-emitter/)
- [Nodejs doc api](https://nodejs.dev/fr/api/v19/events/)
- [Documentation events](https://nodejs.org/api/events.html#events)
- [Nodejs Dependencies](https://nodejs.org/en/docs/meta/topics/dependencies)
- [libvu](https://libuv.org)


---
layout: cover
class: text-center
---

# Typescript EventEmitter

```ts
type Events = { ["myEvent"]: (event: string) => Promise<void>; }
```

<br>

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/4c/Typescript_logo_2020.svg/240px-Typescript_logo_2020.svg.png" class="m-auto h-40" />

---

# Typage EventEmitter

Typage partielle de la classe EventEmitter

```ts
type ListenerSignature<L> = {
  [E in keyof L]: (...args: any[]) => any;
}

interface TypedPartialEventEmitter<Events extends ListenerSignature<Events>> {
  on: <E extends keyof Events>(event: E, listener: Events[E]) => this;
  emit: <E extends keyof Events>(event: E, ...args: Parameters<Events[E]>) => boolean;
}
```

---

# Usage exemple

```ts
const Event = Symbol("Event");

type Events = {
  [Event]: (event: number) => Promise<void>;
}

const myEvents = new EventEmitter() as TypedPartialEventEmitter<Events>;

// => type '"hello"' is not assignable to parameter of type 'number'
myEvents.emit(Event, "hello");
```

--
layout: cover
class: text-center
---

# Architecture 3 tiers

<br>

<img src="https://practica.dev/assets/images/3-tiers-fb96effa6ad8f8f08b594f3455628305.png" class="m-auto" />

---
layout: center
class: text-center
---

# Entrypoints

C'est la porte de l'application où les flux commencent et les demandes arrivent. Notre exemple de composant a une API REST (c'est-à-dire des contrôleurs d'API), c'est un type de point d'entrée. Il peut y avoir d'autres points d'entrée comme une tâche planifiée, une CLI, une file d'attente de messages, etc. Quel que soit le point d'entrée avec lequel vous traitez, la responsabilité de cette couche est minime - recevoir les demandes, effectuer l'authentification, transmettre la demande au code interne et gérer les erreurs. Par exemple, un contrôleur reçoit une demande d'API, puis il ne fait rien de plus que d'authentifier l'utilisateur, d'extraire la charge utile et d'appeler une fonction de couche de domaine.

---
layout: center
class: text-center
---

# Domain

Un dossier contenant le cœur de l'application où les flux, la logique et la structure des données sont définis. Ses fonctions peuvent desservir n'importe quel type de points d'entrée - qu'il soit appelé depuis l'API ou la file d'attente de messages, la couche de domaine est indépendante de la source de l'appelant. Le code ici peut appeler d'autres services via HTTP/file d'attente. Il est également probable qu'il récupère et enregistre des informations dans une base de données, pour cela, il appellera la couche d'accès aux données.

---
layout: center
class: text-center
---


# Data-access

L'intégralité de la fonctionnalité et de la configuration de votre interaction avec la base de données est conservée dans ce dossier.

---
layout: cover
class: text-center
---

# Clean architecture

<br>

<img src="https://tech.gojob.com/static/07b2e4403c83a8b377ad14ab3589044c/41704/clean-archi.avif" class="m-auto" />

---
layout: center
class: text-center
---

# Adapater

<img src="https://tech.gojob.com/static/ba97fa41d5304cf9c3fc6171602eea1f/a2baf/adapters-and-ports.avif" class="m-auto" />

---
layout: center
class: text-center
---

# Thank's

<img src="https://user-images.githubusercontent.com/94382341/159370370-cb8a63c8-2a42-413c-a659-2ce5662eecbf.png" class="m-30 h-30" />

[GitHub](https://github.com/stephen-shopopop)
