# gulp-documentation
Documentação de utilização da biblioteca gulp JS para automação de arquivos de front-end. Nessa documentação é ultilzado um projeto em Laravel para a estrutura dos arquivos, mas a lógica é a mesma para qualquer sistema.

### GulpJS (com versionamento de arquivos)
> **1.** Primeiramente é necessário instalar o gulp e suas dependências:

No package.json, insira nas dependências os seguintes pacotes, e depois rode composer update. Também é necessário definir as configurações de versões no package.json.

```javascript
"devDependencies": {
...
	"gulp-babel": "^8.0.0",
	"gulp-changed": "^4.0.2",
	"gulp-clean-css": "^4.3.0",
	"gulp-concat": "^2.6.1",
	"gulp-if": "^3.0.0",
	"gulp-imagemin": "^7.1.0",
	"gulp-postcss": "^8.0.0",
	"gulp-sourcemaps": "^3.0.0",
	"gulp-terser": "^1.2.0",
},
"suaConfig": {
	"scriptsVer": "1.0.0",
  "stylesVer": "1.0.0",
  "bundleVer": "1.0.0"
}
```
> **2.** Após ter essas dependências instaladas. Crie um arquivo `gulpfile.js` na raiz do projeto.

No arquivo `gulpfile.js` inicialmente é necessário fazer os requires das dependências necessárias:

```javascript
const gulp = require('gulp');
const terser = require('gulp-terser');
const sourcemaps = require('gulp-sourcemaps');
const gulpif = require('gulp-if');
const del = require('del');
const babel = require('gulp-babel');
const concat = require('gulp-concat');
const unprefix = require("postcss-unprefix");
const autoprefixer = require('autoprefixer');
const postcss = require('gulp-postcss');
const cleanCSS = require('gulp-clean-css');
const changed = require('gulp-changed');
const imagemin = require('gulp-imagemin');
```

O código começa por incluir as seguintes dependências:

- gulp: A biblioteca principal do Gulp, que fornece a base para a automação de tarefas.
- terser: Uma biblioteca para minificação de JavaScript.
- sourcemaps: Uma biblioteca para geração de mapas de origem para arquivos minificados.
- gulp-if: Uma biblioteca para ajudar a tomar decisões de fluxo de trabalho baseadas em condições.
- del: Uma biblioteca para apagar arquivos e diretórios.
- babel: Uma biblioteca para compilar o código JavaScript usando o Babel.
- concat: Uma biblioteca para concatenar vários arquivos em um único arquivo.
- postcss: Uma biblioteca para processar o CSS com plugins.
- autoprefixer: Um plugin do PostCSS para adicionar automaticamente prefixos CSS a propriedades que precisam deles.
- cleanCSS: Uma biblioteca para minificação de CSS.
- changed: Uma biblioteca para monitorar mudanças em arquivos e diretórios.
- imagemin: Uma biblioteca para otimização de imagens.

Prossiga no arquivo fazendo algumas configurações:

```javascript
let jsDebug = false;
let jsDev = false;
const babelConfig = {
    presets: ['@babel/env'],
    babelrc: false,
};
let cssDebug = false;
let cleanCSSOpts = {};
const processors = [
    unprefix,
    autoprefixer,
];
const packageConfigs = require('./package.json');
const { scriptsVer, stylesVer } = packageConfigs.suaConfig;
```

Após isso defina os arquivos de destino. Esses arquivos serão os que serão chamados no projeto. Exemplo:

```javascript
const destinationFiles = [
    'public/assets/js/lib.min.js',
    `public/assets/js/all-${scriptsVer}.min.js`,
    `public/assets/css/all-${stylesVer}.min.css`,
    'public/assets/css/all.min.css',
    'public/assets/css/amp.min.css',
    'public/assets/js/amp.min.js',
    'public/assets/gallery/js/gallery.min.js',
    'public/assets/gallery/css/gallery.min.css',
];
```
> **3.**  Depois de definir os arquivos de destino, defina os arquivos que serão compilados. Se quiser você, pode separar os arquivos por tipo. Por exemplo, se quiser separar para compilação todos os arquivos de bibliotecas usadas é possível usar da mesma forma, porém chamando em uma const separada. Exemplo:

```javascript
const libsJavascriptFiles = [
    'resources/assets/js/lib/lozad.min.js',
    'resources/assets/js/lib/jquery-latest.min.js',
];
```
Os demais arquivos padrão do seu site podem ser chamados juntos. Exemplo:

```javascript
const defaultJavascriptFiles = [
    'resources/assets/js/helpers.js',
    'resources/assets/js/scripts.js',];
];
```
Esses foram arquivos JS. Da mesma forma pode ser feito para arquivos CSS. Exemplo:

```javascript
const defaultCssFiles = [
    'resources/assets/css/geral.css',
    'resources/assets/css/styles.css',
];
```

> **4.** A partir de agora serão definidas as tasks do seu script. As tasks do gulp são funções que realizam tarefas específicas em um projeto. Cada tarefa é definida com o método gulp.task() e pode ser executada a partir da linha de comando, fornecendo a capacidade de automatizar tarefas repetitivas.

Primeiramente iremos definir as tasks para os arquivos JS. Exemplo:

```javascript
gulp.task('defaultScripts', () =>
  gulp.src(defaultJavascriptFiles)
    .pipe(gulpif(jsDebug || jsDev, sourcemaps.init()))
    .pipe(gulpif(!jsDebug, babel(babelConfig)))
    .pipe(gulpif(!jsDebug && !jsDev, terser()))
    .pipe(concat(`all-${scriptsVer}.min.js`))
    .pipe(gulpif(jsDebug || jsDev, sourcemaps.write('./')))
    .pipe(gulp.dest('public/assets/js'))
);
```

A task 'defaultScripts' é responsável por processar os arquivos JavaScript padrão definidos na variável defaultJavascriptFiles. Aqui está o que cada etapa da tarefa realiza:
1. O método **`gulp.src`** lê os arquivos especificados em **`defaultJavascriptFiles`**.
2. O método **`gulpif`** é usado para verificar se as variáveis **`jsDebug`** ou **`jsDev`** estão habilitadas. Se alguma delas estiver habilitada, o método **`sourcemaps.init()`** é chamado para inicializar a geração de sourcemaps.
3. O método **`gulpif`** é usado novamente para verificar se a variável **`jsDebug`** está desabilitada. Se estiver desabilitada, o método **`babel`** é chamado para transpilar o código JavaScript para uma versão compatível com diferentes navegadores, usando as configurações definidas em **`babelConfig`**.
4. O método **`gulpif`** é usado novamente para verificar se as variáveis **`jsDebug`** e **`jsDev`** estão desabilitadas. Se ambas estiverem desabilitadas, o método **`terser`** é chamado para minificar o código JavaScript.
5. O método **`concat`** é chamado para concatenar os arquivos processados em um único arquivo, usando o nome **`all-${scriptsVer}.min.js`**.
6. O método **`gulpif`** é usado mais uma vez para verificar se as variáveis **`jsDebug`** ou **`jsDev`** estão habilitadas. Se alguma delas estiver habilitada, o método **`sourcemaps.write`** é chamado para escrever os sourcemaps gerados.
7. O método **`gulp.dest`** é chamado para escrever o arquivo processado na pasta de destino especificada, que é 'public/assets/js'.

Em resumo, a task 'defaultScripts' lê arquivos JavaScript, transpila-os, minifica-os, concatena-os em um único arquivo e escreve o arquivo resultante na pasta de destino, com opções adicionais para gerar sourcemaps.

Defina uma task para cada tag sua definida. Por exemplo, você pode fazer o mesmo para os arquivos definidos na task ‘libScripts’.

Defina uma task para executar as tarefa usando a função `gulp.parallel()` :

```javascript
gulp.task('scripts', (done) =>
gulp.parallel([
    'libScripts',
    'defaultScripts',
  ])(done)
);
```

A tarefa scripts basicamente é uma abreviação para executar a tarefa defaultScripts, sem precisar definir as variáveis jsDev ou jsDebug.

> **5.** Agora você irá configurar algumas tasks para o ambiente de desenvolvimento:

```javascript
gulp.task('scripts:dev', (done) => {
    jsDev = true;
    return gulp.parallel([
        'libScripts',
        'defaultScripts',
    ])(done);
});

gulp.task('scripts:debug', (done) => {
    jsDebug = true;
    return gulp.parallel([
        'libScripts',
        'defaultScripts',
    ])(done);
});
```

Estas duas tarefas, **`scripts:dev`** e **`scripts:debug`**, são tarefas personalizadas do gulp que servem como uma interface para a tarefa **`defaultScripts`**.

A tarefa **`scripts:dev`** atribui o valor **`true`** à variável **`jsDev`** e, em seguida, executa a tarefa **`defaultScripts`** em paralelo.

A tarefa **`scripts:debug`** atribui o valor **`true`** à variável **`jsDebug`** e, em seguida, executa a tarefa **`defaultScripts`** em paralelo.

Com isso, as tarefas **`scripts:dev`** e **`scripts:debug`** servem como uma maneira de controlar se a tarefa **`defaultScripts`** está sendo executada no modo de desenvolvimento ou modo de debug. Estas flags provavelmente afetarão a forma como a tarefa **`defaultScripts`** é executada, por exemplo, produzindo saídas de depuração ou permitindo que o código não otimizado seja produzido.

Você deve fazer o mesmo para os arquivos CSS:

```javascript
gulp.task('defaultCSS', () =>
  gulp.src(defaultCssFiles)
    .pipe(gulpif(!cssDebug, postcss(processors)))
    .pipe(gulpif(!cssDebug, cleanCSS(cleanCSSOpts)))
    .pipe(concat(`all-${stylesVer}.min.css`))
    .pipe(gulp.dest('public/assets/css'))
);
```

Faça o mesmo para todos os arquivos CSS, caso tenha alguma divisão como as de biblioteca no JS.

Após isso defina uma task ‘css’ da mesma forma como foi feito nos arquivos JS.

```javascript
gulp.task('css', (done) =>
  gulp.parallel([
		'defaultCSS',
			 ...
	])(done)
);
```

Esse trecho de código define tarefas relacionadas ao processamento de arquivos CSS. A tarefa **`defaultCSS`**é responsável por processar os arquivos CSS padrão especificados na variável **`defaultCssFiles`**. Esses arquivos são então passados por uma série de processos de minificação, concatenação e outros, de acordo com as condições especificadas com a função **`gulpif`**.

Após isso defina as confgurações de desenvolvimento e depuração:

```javascript
gulp.task('css:dev', (done) => {
    cleanCSSOpts = { format: 'beautify', level: 2 };
    return gulp.parallel(['defaultCSS', ...])(done);
});

gulp.task('css:debug', (done) => {
    cssDebug = true;
    return gulp.parallel(['defaultCSS', ...])(done);
});
```

> **6.** Depois de todas as tarefas definidas, configure as tasks clean e watch:
```javascript
gulp.task('clean', () =>
    del(destinationFiles)
);
gulp.task('watch', function () {
    gulp.watch([
	    ...defaultJavascriptFiles,
	    ],
	    gulp.series(['scripts:debug'])
	    );
	    gulp.watch([
	    ...defaultCssFiles
	    ]
	    ,
	    gulp.series(['css:debug'])
  );
});
```

A tarefa **`clean`** é usada para apagar arquivos em um destino específico, indicado no array **`destinationFiles`**. Isso pode ser útil para garantir que os arquivos gerados sejam sempre atualizados, sem deixar arquivos desatualizados ou desnecessários no destino.

A tarefa **`watch`** é usada para vigiar as alterações em arquivos específicos e atualizar o build automaticamente. Quando algum arquivo em **`defaultJavascriptFiles`** for alterado, a tarefa **`scripts:debug`** será executada. Da mesma forma, quando algum arquivo em **`defaultCssFiles`** for alterado, a tarefa **`css:debug`** será executada. Isso facilita o desenvolvimento, pois evita que o desenvolvedor tenha que rodar manualmente a tarefa toda vez que houver uma alteração.

> **7.** Defina os comandos que serão executados ao acionar o gulp no console:

```javascript
exports.default = gulp.series([
'clean',
    gulp.parallel([
    'scripts',
    'css'
    ])
]);
exports.dev = gulp.series([
'clean',
    gulp.parallel([
    'scripts:dev',
    'css:dev'
    ])
]);
exports.debug = gulp.series([
'clean',
    gulp.parallel([
    'scripts:debug',
    'css:debug'
    ])
]);
exports.watch = gulp.series(['watch']);
```

**`exports.default`** define a tarefa padrão que é executada ao digitar **`gulp`** no terminal. Essa tarefa consiste em primeiro limpar os arquivos existentes no diretório de destino (através da tarefa **`clean`**) e depois processar todos os arquivos CSS e JavaScript em paralelo (através das tarefas **`css`** e **`scripts`**).

**`exports.dev`** define uma tarefa específica para desenvolvimento. A tarefa consiste em primeiro limpar os arquivos existentes no diretório de destino (através da tarefa **`clean`**) e depois processar todos os arquivos CSS e JavaScript em paralelo usando a versão de desenvolvimento (através das tarefas **`css:dev`** e **`scripts:dev`**).

**`exports.debug`** define uma tarefa específica para depuração. A tarefa consiste em primeiro limpar os arquivos existentes no diretório de destino (através da tarefa **`clean`**) e depois processar todos os arquivos CSS e JavaScript em paralelo usando a versão de depuração (através das tarefas **`css:debug`** e **`scripts:debug`**).

**`exports.watch`** define uma tarefa que permite acompanhar as alterações nos arquivos CSS e JavaScript em tempo real e executar as tarefas adequadas a cada alteração (**`css:debug`** ou **`scripts:debug`**).

> **8.** No console da aplicação: rode o comando **`gulp`** para executar o script ou **`gulp watch`** para deixar o build vigiando as alterações nos arquivos.

Os arquivos serão compilados com a versão definida no package.json. Dessa forma é necessário fazer a chamada certa no HTML para usar a última versão compilada. Já que estamos utilizando Laravel, crie um arquivo de configuração na pasta **config**. Chamaremos de **aliases.php** mas você pode dar o nome que preferir.

Dentro do arquivo você irá definir alguns apelidos para serem usados como variáveis globais dentro do seu projeto. Insira o seguinte código:

```php
<?php
$pckJson = file_get_contents(base_path('package.json'));
$pckConfigs = json_decode($pckJson, true);

return [
'scripts_ver' => $pckConfigs['suaConfig']['scriptsVer'],
'styles_ver' => $pckConfigs['suaConfig']['stylesVer'],
'bundle_ver' => $pckConfigs['suaConfig']['bundleVer'],
];
```
Aqui você está pegando a versão dos seus scripts e estilos definidos no **package.json**.

Agora na VIEW, em seu HTML onde chama o css, defina o seguinte:
```html
@php
    $src = 'assets/css/all-'.config("aliases.styles_ver").'.min.css';
@endphp
<link rel="stylesheet" type="text/css" media="all" href="{{ asset($src) }}" />
```

Pronto! Você estará usando a última versão de seus estilos e scripts.










