---
layout: post
title: "Como criar sistemas que evoluem"
date: 2017-10-08 21:00:00 -0300
categories: Arquitetura
author_url: "https://medium.com/@wilcorrea"
description: "Tem algumas coisas que a gente sabe que vão acontecer, estas são as que acontecem por acaso. Todo o resto é o que compõe a certeza."
author: 
  name: "William Correa"
  email: "wilcorrea@gmail.com"
  author_url: "https://wilcorrea.rocks"
twitter: "https://twitter.com/wilcorrea"
facebook: "https://fb.com/wilcorrea.rocks"
telegram: "https://t.me/wilcorrea"
gravatar: true
---
# Como criar sistemas que evoluem

Sempre que alguém vai começar a trabalhar em um projeto precisa escolher ou entender o como aquele projeto será ou foi feito.
Quando isso acontece é comum ter um desafio para o time porque não há um consenso claro sobre padrões a serem seguidos.
É comum surgirem muitos nomes de metodologias e frameworks, mas é totalmente possível seguir metodologias e utilizar frameworks de formas diferentes.

Essa segmentação no mundo PHP sempre foi um impasse até que a iniciativa do PHP-FIG trouxe a luz alguns padrões para algumas coisas. Estamos agora numa nova fase do PHP onde o fluxo de trabalho precisa ser revisto e o PHP da década passada erradicado dos projetos modernos. A imagem abaixo pode ser vista em seu contexto original [aqui](https://developer.mozilla.org/en-US/docs/Learn/Server-side/Express_Nodejs/routes) onde ela é usada para explicar o funcionamento de uma aplicação feita usando Expresse, e pasmem, ela não é muito diferente do que você verá em ferramentas como CakePHP, Laravel, Symfony, Zends, Slim e etc.

![image](https://cdn-images-1.medium.com/max/800/0*VvmaP5PBCo-6n8sU.png)

A web está consolidada. É isso! Achamos nosso jeito de trabalhar. Um projeto em uma linguagem não é mais tão distante do ponto de vista da arquitetura das outras. Sendo assim aqui vão minhas dicas para criar projetos que sejam simples de escalar e fáceis de serem mantidos:

### 1. Arquitetura em primeiro lugar

Não importa o tamanho da sua aplicação. Para ter um controle pleno dela o ideal é que você tenha, no mínimo, uma distribuição como a demonstrada na imagem acima. Mais pra baixo no artigo vou explicar melhor as razões disso, o que é importante entender agora é que as PSR's devem fazer parte da sua vida para você poder ser uma pessoa mais feliz. Além disso, a ideia de separar sua aplicação no formato em que está descrito na imagem é simplesmente muito boa. Você vai poder usufruir de recursos como middlewares, distribuir melhor suas classes, esclarecer vários conceitos de organização e etc. Do que você precisa para fazer isso? A maioria dos grandes frameworks de mercado seguem esse padrão, mas vamos ver a seguir princípios básicos que podem ajudar a entender como e porque essas ferramentas funcionam.

#### 1.1. Routes (Rotas)

As rotas simplificam o acesso aos seus arquivos. Se até ondem o navegador acessava cada arquivo que você criava, usando um sistema de rotas isso muda. O arquivo não precisa saber mais nada sobre quem está logado ou fazer includes e requires, vão ter outras camadas responsáveis por isso no projeto

Bad
```php
?>
<a href="<?php echo '/registration/product-list.php' ?>">
<a href="<?php echo '/registration/product-edit.php?id=1' ?>">
```

Good
```php
?>
<a href="<?php route('/registration/products') ?>">
<a href="<?php route('/registration/products/1') ?>">
```

#### 1.2. Controllers (Controladores)

Os controladores são classes que capturam o que vem do HTTP e delegam para a aplicação resolver. Devem ser pequenos e ter objetivos direcionados. Quando um controller precisar resolver muita coisa você pode criar um repositório e utilizar ele. Isso evitará a poluição do seu model.

Bad
```php
class Product
{
    public function index()
    {
        echo '<table></table>';
    }
}
```

Good
```php
namespace App\Registration\Controller;
namespace Core\Http\ViewController;

class Product extends ViewController
{
    public function index()
    {
        return $this->view('registration/product/index', []);
    }
}
```

#### 1.3. View (Visualização)

Quando você redeber um request HTTP e precisar realizar uma impressão de uma página você poderá deixar toda a parte de visualização a critério desse camada. Os controllers precisarão ter a habilidade de invocar a sua `template engine` para mesclar os valores que você precisar plotar com o arquivo que tem a formatação de saída (template).

#### 1.4. Model (Modelo)

Os models hoje tem tipo um papel cada vez mais representativo em relação a composição de rotinas, oferecendo alguns metada-dados para os ORM's aos quais costumam estar conectados poderem trabalhar com mais facilidade e implementando rotinas de manipulação e tranformação da informação. Também é comum ver os models como objetos de dados, ou sendo usados através de anotações ou implementações mais complexas para criar processamentos e rotinas para processamentos de ORM's mais robustos.

#### 1.5. Database (Bancos de Dados)

Já dei uma palhinha acima, mas para esclarecer melhor essa camada é responsável pela persistência. É o ponto onde a aplicação sai do escopo da linguagem e precisa ir no disco para persistir os dados. A parte de database da sua aplicação provavelmente vai gerar os comandos SQL em tempo de execução (deixando os DBA's inconformados) e através desses 

### 2. Centralize ações. 

Priorize criar funções e/ou classes para tudo o que for uma operação comum. Se quiser um nome para estudar pode pesquisar por DRY ou KISS que são conceitos que forjam essas ideias de não repetir trechos nem operações. Encapsular relações com o sistema é sempre útil.

#### 2.1. Acesso ao HTTP
Bad
```php
$var = $_POST['var'];
```

Good
```php
$var = post('var');
```

#### 2.2. Gerenciamento de estados
Bad
```php
if (isset($_SESSION['conected'])) {
    header('Location: index.php');
}
```

Good
```php
if (!Auth::isConnected()) {
    Request::redirect('/');
}
```

#### 2.3. Controllers Gordos
Bad
```php
function saveAllItems(Request $request, Validator $validator, Product $product, Item $item)
{
    $rules = [
      'field' => 'required', 'column' => 'required', 'treta' => 'required', 'whatever' => 'required'
    ];
    foreach($request->getBody as $key => $value) {
      if (isset($rules[$key]) && !$validator->validate($value, $rules[$key])) {
        throw new ValidationException($value, $rules[$key]);
      }
    }
    $id = $product->save([
      $request->post('name'),
      $request->post('value'),
    ]);

    foreach($request->post('items') as $productItem) {
      $item->save([
        $request->post('name'),
        $productItem,
      ]);
    }
    
    return true
}
```

Good
```php
function saveAllItems(Request $request, Product $repository)
{
    return $repository->saveAllItems($request->getBody)
}
```
