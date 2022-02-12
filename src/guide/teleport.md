# Teleporte

<VideoLesson href="https://vueschool.io/lessons/vue-3-teleport?friend=vuejs" title="Aprenda a usar o teleport com Vue School">Aprenda como usar o teleporte em uma aula grátis do Vue School</VideoLesson>

O Vue nos encoraja à construir nossas interfaces de usuário (UIs) encapsulando a UI e seu respectivo comportamento em componentes. Podemos aninhar componentes dentro de outros componentes e montar uma árvore que constitui a UI da aplicação.

Entretanto, algumas vezes uma parte do _template_ do componente pertence à esse componente de forma lógica, enquanto do ponto de vista técnico, seria preferível mover essa parte do _template_ para outro lugar no DOM, fora da aplicação Vue.

Um cenário comum para isso é a criação de um componente que inclui um _modal_ em tela cheia. Na maioria dos casos, você gostaria que a lógica do modal residisse dentro do componente, mas o posicionamento do modal se torna difícil de resolver por meio de CSS ou requer uma mudança na composição do componente.

Considere a seguinte estrutura HTML.

```html
<body>
  <div style="position: relative;">
    <h3>Tooltips com Vue 3 e Teleport</h3>
    <div>
      <modal-button></modal-button>
    </div>
  </div>
</body>
```

Vamos olhar o `modal-button` de perto.

O componente terá um elemento `button` para acionar a abertura do modal e um elemento `div` com uma classe `.modal` que irá conter o conteúdo do modal e um botão para fechá-lo.

```js
const app = Vue.createApp({});

app.component('modal-button', {
  template: `
    <button @click="modalOpen = true">
        Abrir modal de tela cheia!
    </button>

    <div v-if="modalOpen" class="modal">
      <div>
        Eu sou um modal! 
        <button @click="modalOpen = false">
          Fechar
        </button>
      </div>
    </div>
  `,
  data() {
    return {
      modalOpen: false
    }
  }
})
```

Ao usar esse componente dentro da estrutura HTML inicial, podemos ver um problema - o modal está sendo renderizado dentro de uma `div` profundamente aninhada e a `position: absolute` do modal toma como referência a `div` pai posicionada relativamente. 

O Teleporte fornece uma maneira limpa para nos permitir controlar sob que elemento pai no DOM nós queremos que uma parte do HTML seja rederizada, sem ter que recorrer ao estado global ou dividí-lo em dois componentes.

Vamos modificar nosso `modal-button` para usar o `<teleport>` e dizer ao Vue que "**teleporte** esse HTML **para** a _tag_ '**body**'".

```js
app.component('modal-button', {
  template: `
    <button @click="modalOpen = true">
        Abrir modal de tela cheia! (Com Teleporte!)
    </button>

    <teleport to="body">
      <div v-if="modalOpen" class="modal">
        <div>
          Eu sou um modal teleportado!
          (Meu pai é o "body")
          <button @click="modalOpen = false">
            Fechar
          </button>
        </div>
      </div>
    </teleport>
  `,
  data() {
    return {
      modalOpen: false
    }
  }
})
```

Como resultado, uma vez que clicamos no botão para abrir o modal, o Vue irá renderizar corretamente o conteúdo como um filho da _tag_ `body`.

<common-codepen-snippet title="Vue 3 Teleport" slug="gOPNvjR" tab="js,result" />

## Usando com Componentes Vue

Se o `<teleport>` contém um componente Vue, o componente permanecerá um filho lógico do pai do `<teleport>`:

```js
const app = Vue.createApp({
  template: `
    <h1>Instância raiz</h1>
    <parent-component />
  `
})

app.component('parent-component', {
  template: `
    <h2>Este é um componente pai</h2>
    <teleport to="#endofbody">
      <child-component name="John" />
    </teleport>
  `
})

app.component('child-component', {
  props: ['name'],
  template: `
    <div>Olá, {{ name }}</div>
  `
})
```

Neste caso, mesmo quando o `child-component` é renderizado em um lugar diferente, permanecerá como filho do `parent-component` e receberá a propriedade `name` dele.

Isso também significa que injeções vindas do componente pai funcionam como o esperado, e que o componente filho será aninhado embaixo do componente pai no _Vue Devtools_, em vez de ser colocado no lugar para o qual o conteúdo foi movido.

## Usando Múltiplos Teleportes no Mesmo Alvo

Um cenário de caso de uso comum pode ser um componente `<Modal>` reutilizável, do qual pode haver várias instâncias ativas ao mesmo tempo. Para esse tipo de cenário, múltiplos componentes `<teleport>` podem montar seus respectivos conteúdos no mesmo elemento alvo. A ordem será simplemente anexá-los - as montagens novas serão localizadas depois das primeiras montagens realizadas no elemento alvo.

```html
<teleport to="#modals">
  <div>A</div>
</teleport>
<teleport to="#modals">
  <div>B</div>
</teleport>

<!-- result-->
<div id="modals">
  <div>A</div>
  <div>B</div>
</div>
```

Você pode checar as opções do componente `<teleport>` nas [referências da API](../api/built-in-components.html#teleport).
