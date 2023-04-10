---
theme: seriph
titleTemplate: '%s - EventEmitter'
# enable presenter mode, can be boolean, 'dev' or 'build'
presenter: true
download: true
# enable slide recording, can be boolean, 'dev' or 'build'
record: 'build'
exportFilename: eventEmitter-exported
export:
  format: pdf
  timeout: 30000
  dark: false
  withClicks: false
  withToc: false
background: https://source.unsplash.com/collection/298137/1920x1080
class: text-center
highlighter: shiki
lineNumbers: true
info: |
  ## Team's Shopopop
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
drawings:
  persist: false
fonts:
  sans: Robot
  serif: Robot Slab
  mono: Fira Code
title: Welcome to team's Shopopop
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

---
layout: cover
class: text-center
---

# Architecture 3 tiers

<br>

<img src="/74C38AFF-F646-4ED0-A9A6-D85DF56062E5.png" class="m-auto" />

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
layout: center
class: text-center
---

# Organisez votre projet en composants

Le pire obstacle des énormes applications est la maintenance d'une base de code immense contenant des centaines de dépendances - un tel monolithe ralentit les développeurs tentant d'ajouter de nouvelles fonctionnalités. Pour éviter cela, répartissez votre code en composants, chacun dans son dossier avec son code dédié, et assurez vous que chaque unité soit courte et simple.

Autrement : Lorsque les développeurs qui codent de nouvelles fonctionnalités ont du mal à réaliser l'impact de leur changement et craignent de casser d'autres composants dépendants - les déploiements deviennent plus lents et plus risqués. Il est aussi considéré plus difficile d'élargir un modèle d'application quand les unités opérationnelles ne sont pas séparées.

---
layout: center
class: text-center
---

# Organisez vos composants en strates, gardez la couche web à l'intérieur de son périmètre

Chaque composant devrait contenir des « strates » - un objet dédié pour le web, un pour la logique et un pour le code d'accès aux données. Cela permet non seulement de séparer clairement les responsabilités mais permet aussi de simuler et de tester le système de manière plus simple. Bien qu'il s'agisse d'un modèle très courant, les développeurs d'API ont tendance à mélanger les strates en passant l'objet dédié au web (Par exemple Express req, res) à la logique opérationnelle et aux strates de données - cela rend l'application dépendante et accessible seulement par les frameworks web spécifiques.

Autrement : Les tests, les jobs CRON, les déclencheurs des files d'attente de messages et etc ne peuvent pas accéder à une application qui mélange les objets web avec les autres strates.

---
layout: center
class: text-center
---

# Externalisez les utilitaires communs en paquets NPM

Dans une grande appli rassemblant de nombreuses lignes de codes, les utilitaires opérant sur toutes les strates comme un logger, l'encryption et autres, devraient être inclus dans le code et exposés en tant que paquets NPM privés. Cela permet leur partage au sein de plusieurs projets.

Autrement : Vous devrez inventer votre propre roue de déploiement et de dépendance

---
layout: cover
class: text-center
---

# Clean architecture

<br>

<img src="/356E80C1-ABBE-4D95-B05F-4848F9966C11.jpeg" class="m-auto h-60" />

---
layout: center
class: text-center
---

# Adapters

<img src="/0E82E760-E764-4B5A-87B1-882F5D0AD736.jpeg" class="m-auto" />

---
layout: center
class: text-center
---

# Thank's

<img src="https://user-images.githubusercontent.com/94382341/159370370-cb8a63c8-2a42-413c-a659-2ce5662eecbf.png" class="m-30 h-30" />

[GitHub](https://github.com/stephen-shopopop)
