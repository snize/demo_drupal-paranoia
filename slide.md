# Drupal Paranoiaで<br>Drupalをより安全に

金子 智嗣
[@snize](https://twitter.com/snize)<br>
kaneko@zerobase.jp<br>


## Drupal標準の公開ディレクトリ構成


```
❯ tree web -L 1
web
├── autoload.php
├── core
├── index.php
├── modules
├── profiles
├── robots.txt
├── sites
├── themes
├── update.php
└── web.config

5 directories, 5 files
```


## Drupal Paranoia

Composerのプラグイン
Note:
**利用の前提として、変更するDrupalプロジェクトが[drupal-composer/drupal-project](https://github.com/drupal-composer/drupal-project)で作成されていること。**


## 何をするプラグインなのか

Drupalのディレクトリ構成を変更をサポート
Note:
具体的には**web**ディレクトリを**app**へリネーム後、**composer drupal:paranoia**を実行または、初回であれば**composer require drupal-composer/drupal-paranoia:~1**実行時にリネーム先のphpファイルを読み込むphpファイルを生成し、js, css, imagesは移動先へシンボリックリンクの生成を行う


### なぜこのプラグインを使うか

ウェブ側からアクセスできる場所にあるファイルから**<br>脆弱性を生まないようにする。**
Note:実際に[Coder](https://www.drupal.org/project/coder)というモジュールで2016年に脆弱性[SA-CONTRIB-2016-039](https://www.drupal.org/node/2765575)が発生した。

> The module does not need to be enabled for this to be exploited. Its presence on the file system and being reachable from the web are sufficient.



## 確認



### 変更前

```
composer create-project drupal-composer/drupal-project:8.x-dev some-dir --stability dev --no-interaction
```


```
❯ tree -L 1
.
├── LICENSE
├── README.md
├── composer.json
├── composer.lock
├── drush
├── load.environment.php
├── phpunit.xml.dist
├── scripts
├── vendor
└── web

4 directories, 6 files
```


```
❯ tree web -L 1
web
├── autoload.php
├── core
├── index.php
├── modules
├── profiles
├── robots.txt
├── sites
├── themes
├── update.php
└── web.config

5 directories, 5 files
```



## Drupal Paranoiaで変更を加える

```
mv web app
composer require drupal-composer/drupal-paranoia:~1
```


## 実行後


webをappにリネーム。drupal paranoiaによってwebが生成
```
❯ tree -L 1
.
├── LICENSE
├── README.md
├── app
├── composer.json
├── composer.lock
├── drush
├── load.environment.php
├── phpunit.xml.dist
├── scripts
├── vendor
└── web

5 directories, 6 files
```


drupal paranoiaによって生成されたファイル
```
❯ tree web -L 1
web
├── core
├── index.php
└── sites

2 directories, 1 file
```


リネームのみなので、デフォルトのwebと中身は一緒
```
❯ tree app -L 1
app
├── autoload.php
├── core
├── index.php
├── modules
├── profiles
├── robots.txt
├── sites
├── themes
├── update.php
└── web.config

5 directories, 5 files
```


webの中にはほぼPHPファイルなし
```
❯ find ./web -name "*.php"
./web/core/install.php
./web/core/modules/statistics/statistics.php
./web/core/rebuild.php
./web/index.php
```

```
❯ cat ./web/index.php
<?php

chdir('../app/');
require './index.php';
```

```
❯ cat ./web/core/install.php
<?php

chdir('../../app/core/');
require './install.php';
```


coreやsite以下もファイルの実体は無くシンボリックリンクのみ
```
❯ tree web/core/themes/classy/css
web/core/themes/classy/css
└── components
    ├── action-links.css -> ../../../../../../app/core/themes/classy/css/components/action-links.css
    ├── book-navigation.css -> ../../../../../../app/core/themes/classy/css/components/book-navigation.css
    ├── breadcrumb.css -> ../../../../../../app/core/themes/classy/css/components/breadcrumb.css
    ├── button.css -> ../../../../../../app/core/themes/classy/css/components/button.css
    ├── collapse-processed.css -> ../../../../../../app/core/themes/classy/css/components/collapse-processed.css
    ├── container-inline.css -> ../../../../../../app/core/themes/classy/css/components/container-inline.css
    ├── details.css -> ../../../../../../app/core/themes/classy/css/components/details.css
    （略）
```


### 動作確認

```
drush site:install demo_umami --db-url="sqlite://sites/default/files/.ht.sqlite" -y
cd web && php -S localhost:8888
```



## まとめ

今回の場合は`web(DocumentRoot)`以下が必要最小限のファイルに置き換えられ、外部のアクセスから守られる。結果としては、PHP製のウェブフレームワークのディレクトリ構造に近くなった。

- [Symfony Quick Tour: The Architecture](https://symfony.com/doc/3.4/quick_tour/the_architecture.html)
- [Directory Structure - Laravel - The PHP Framework For Web Artisans](https://laravel.com/docs/5.6/structure)


## 以上です、ありがとうございました<br /> (｡･ω･｡)ﾉ


#### コンタクト

- [@snize](https://twitter.com/snize)
- @tomotsugu_kaneko - [drupal-japan.slack.com](https://docs.google.com/forms/d/e/1FAIpQLSeuP5tW2Rrwte0c2VNkZnEJ_lTKNRRUicqBbR1S7wUmbqox2A/viewform?c=0&w=1)
- kaneko@zerobase.jp


## おまけ

最近[DrupalCon Nashville2018](https://events.drupal.org/nashville2018)以降、Drupal周辺のComposerプロジェクトが活発になっている気がする。

- [Proposal: Composer Support in Core initiative [#2958021] | Drupal.org](https://www.drupal.org/project/ideas/issues/2958021)
- [Automatic Updates initiative [#2940731] | Drupal.org](https://www.drupal.org/project/ideas/issues/2940731)