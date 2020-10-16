FROM php:7.4-fpm

#================================================#
# NOTE:
#   - e: パイプやサブシェルで実行したコマンドが1つでもエラーになったら直ちにシェルを終了する
#   - u: パラメーター展開中に、設定していない変数があったらエラーとする
#   - x: トレース情報として、シェルが実行したコマンドとその引数を出力する
#================================================#
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
  #================================================#
  # NOTE:
  #   - ghostscript: PDFプレビューの表示に必要らしい
  #================================================#
  ghostscript \
  ; \
  rm -rf /var/lib/apt/lists/*

#================================================#
# NOTE:
#   - WPに必要なPHPのライブラリをインストールする
#   - https://make.wordpress.org/hosting/handbook/handbook/server-environment/#php-extensions
#   - savedAptMark: 手動でインストールしたパッケージを事前に保存しておいて、これからインストールするビルドにしか使わないパッケージだけ後で削除するために使う
#================================================#
RUN set -ex; \
  \
  savedAptMark="$(apt-mark showmanual)"; \
  \
  apt-get update; \
  apt-get install -y --no-install-recommends \
  libfreetype6-dev \
  libjpeg-dev \
  libmagickwand-dev \
  libpng-dev \
  ; \
  \
  #================================================#
  # NOTE:
  #   - docker-php-ext-configure: PHP拡張のカスタマイズ
  #   - docker-php-ext-install: PHP拡張を解凍してインストール
  #   - gd: 画像処理に使う
  #   - bcmath: 桁数に多い数字の計算に使う
  #   - exif: 画像のメタデータ処理に使う
  #   - mysqli: MySQLへの接続に使う
  #   - zip: プラグイン・テーマ・WPのアップデートパッケージの解凍に使う
  #   - imagick: 画像の品質関係に使う
  #================================================#
  docker-php-ext-configure gd --with-freetype-dir=/usr --with-jpeg-dir=/usr --with-png-dir=/usr; \
  docker-php-ext-install -j "$(nproc)" \
  bcmath \
  exif \
  gd \
  mysqli \
  zip \
  ; \
  pecl install imagick-3.4.4; \
  docker-php-ext-enable imagick; \
  \
  #================================================#
  # NOTE:
  #   - ビルドの依存関係を全て削除する
  #   - ldd: 指定したファイルで使っている共有ライブラリを表示する
  #   - awk: 空白などで区切られたテキストを処理する
  #   - sort -u: 重複を削除する
  #   - xargs: 標準入力やファイルからリストを受け取って、他のコマンドの引数として使う
  #   - dpkg-query: インストール済みパッケージの情報を調べる
  #   - cut: 指定した部分だけを切り出す。
  #================================================#
  apt-mark auto '.*' > /dev/null; \
  apt-mark manual $savedAptMark; \
  ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
  | awk '/=>/ { print $3 }' \
  | sort -u \
  | xargs -r dpkg-query -S \
  | cut -d: -f1 \
  | sort -u \
  | xargs -rt apt-mark manual; \
  \
  apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
  rm -rf /var/lib/apt/lists/*
