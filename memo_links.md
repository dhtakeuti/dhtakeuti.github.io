# JSF

- 動的な画面の表示｜Jakarta EE チュートリアル [[Link](https://zenn.dev/atsushi_ni/books/fc0704403f28db/viewer/27eac4)]

- Jakarta EEをはじめよう！ #JavaEE - Qiita [[Link](https://qiita.com/tkxlab/items/07083168be6400d71fd4)]



# Java HttpClient 再利用

- 今すぐ使える！Java HttpClientの基本操作と活用例まとめ [[Link](https://code-rebuild.com/java-for-beginners/java-httpclient/)]

  - ベストプラクティスの例
    ```java
    // アプリケーションで共有するHttpClientインスタンス
    public class AppHttpClient {
      public static final HttpClient INSTANCE = HttpClient.newBuilder()
            .version(HttpClient.Version.HTTP_2)
            .connectTimeout(Duration.ofSeconds(10))
            .build();
    
      private AppHttpClient() {}
    }
    
    // 利用する側のコード
    public class SomeService {
      public void doSomething() throws Exception {
        HttpRequest request = ...;
        // 共有インスタンスを使ってリクエストを送信
        HttpResponse<String> response = AppHttpClient.INSTANCE
          .send(request, HttpResponse.BodyHandlers.ofString());
      }
    }
    ```

- 並列処理で発生したスレッド増加問題と解決策 - RAKUS Developers Blog | ラクス エンジニアブログ [[Link](https://tech-blog.rakus.co.jp/entry/20250918/resttemplate-thread-leak-fix)]

  - シングルトンで再利用するケース (Spring の RestTemplate)
    ```java
    @Configuration
    class RestTemplateConfig {
      @Bean
      fun restTemplate(builder: RestTemplateBuilder): RestTemplate {
          return builder
            .setConnectTimeout(Duration.ofSeconds(5))
            .setReadTimeout(Duration.ofSeconds(30))
            .build()
      }
    }
    
    @Service
    class OcrService(
      private val restTemplate: RestTemplate) {
        fun callOcrApi(request: OcrRequest): OcrResponse {
          return restTemplate.postForObject("https://example.com/ocr",
                                            request, OcrResponse::class.java)!!
        }
    }
    ```

- AI の回答

- Javaの `HttpClient`（特にJava 11以降の標準API）は、**インスタンスを再利用する（使い回す）のがベストプラクティス**です。

  なぜ再利用すべきなのか

  - **接続の再利用（Keep-Alive）**: `HttpClient`を再利用することで、同一ホストへのTCP接続がプールされ、リクエストごとにコネクションを張り直すオーバーヘッドがなくなります。
  - **リソース消費の抑制**: 都度インスタンスを生成すると、使用済みのソケットが`TIME_WAIT`状態で残り続け、大量の通信を行う際に**ポート枯渇（Socket Exhaustion）**を引き起こすリスクがあります。
  - **スレッド管理**: `HttpClient`は内部的に独自のスレッドプールを管理しているため、頻繁に生成・破棄を繰り返すとリソース管理の効率が悪化します。 

  推奨される実装方法

  通常、アプリケーション全体で1つのインスタンスを保持し、それをスレッドセーフに共有して使用します。

  ```java
  public class MyService {
    // staticフィールドやシングルトンとして保持する
    private static final HttpClient client = HttpClient.newBuilder()
          .version(HttpClient.Version.HTTP_2) // HTTP/2対応
          .connectTimeout(Duration.ofSeconds(10))
          .build();
  
    public void sendRequest() throws Exception {
        HttpRequest request = HttpRequest.newBuilder()
              .uri(URI.create("https://example.com"))
              .build();
          
        // 同じclientを何度も呼び出す
        HttpResponse<String> response = client.send(
          request, HttpResponse.BodyHandlers.ofString());
      }
  }
  ```

  **注意点**

  - **不変性**: `HttpClient`インスタンスは作成後に変更できません。異なる設定（プロキシ、タイムアウトなど）が必要な場合は、別のインスタンスを作成するか、`Builder`をコピーして構築します。
  - **他ライブラリの場合**:
    - **Apache HttpClient**: `CloseableHttpClient`を使い回し、`PoolingHttpClientConnectionManager`で接続を管理するのが一般的です。
    - **RestTemplate (Spring)**: 内部で使っている`ClientHttpRequestFactory`を適切に設定し、接続プールを有効にする必要があります。 

  再利用する際、**特定のタイムアウト設定やプロキシ設定**などをリクエストごとに切り替える必要があるでしょうか？

- Apache HttpClient 5.x 再利用(使いまわし) 接続設定 ポイント まとめ #Java - Qiita [[Link](https://qiita.com/gosshys/items/df1ea26ba2e8c0860ae4)]

- 障害を回避するHttpClient再入門 / Avoiding Failures HttpClient Reintroduction - Speaker Deck [[Link](https://speakerdeck.com/uskey512/avoiding-failures-httpclient-reintroduction)]
  - リクエスト全体を制限するタイムアウトを `HttpRequest.timeout` で設定するべし。

- Java HTTP Client API のタイムアウト設定 #Java - Qiita [[Link](https://qiita.com/niwasawa/items/5db7180f7faa7545cf8b)]

- java.net.http.HttpClient にて、IOException: Too many open files エラーが発生する場合の対応 #Java - Qiita [[Link](https://qiita.com/huge-book-storage/items/b88d7061fb7c1e6dfde9)]



# Java HttpClient で TLS

- Java11 HttpClient SSL サンプル - Oboe吹きプログラマの黙示録 [[Link](https://oboe2uran.hatenablog.com/entry/2022/11/26/133927)]

- 【Java】HttpClientでAPIを叩いてみる① #HTTP - Qiita [[Link](https://qiita.com/syoshika_/items/dd1d699aed7ffe1d36e2)]

- 【Java】HttpClientでAPIを叩いてみる② #HTTP - Qiita [[Link](https://qiita.com/syoshika_/items/e90a6acd9335a2380db8)]

- https 証明書 java21 #HTTPS - Qiita [[Link](https://qiita.com/kaidanouji1984/items/953841431096518ea65f)]
  - 読みにくい、text ファイルにしたら読みやすくなるかも



- Let's EncryptでHTTPSを終端させたいだけならNginxよりCaddyを使うと楽だった件 #Go - Qiita [[Link](https://qiita.com/ssc-ksaitou/items/ee0cda84dcf358a2b5eb)]





# 単体テスト

- 10年稼働するコードを作る『単体テストの考え方/使い方』完全要約 [[Link](https://code-rebuild.com/it-general/unit-testing-concepts/)]

- Mockitoを使いこなす！Java単体テストを効率化する完全ガイド - momuki [[Link](https://omomuki-tech.com/archives/6445)]

- JUnitで学ぶ実践的で本質的なユニットテストの考え方 | 株式会社一創 [[Link](https://www.issoh.co.jp/tech/details/3064/)]

- PowerMockitoを使用した高度なMockオブジェクトの宣言方法 | 株式会社一創 [[Link](https://www.issoh.co.jp/tech/details/2351/)]

- PowerMockitoの概要と利用シーン：モック化フレームワークの基本 | 株式会社一創 [[Link](https://www.issoh.co.jp/tech/details/3637/)]





# Java スレッド

- Java並行処理の神髄！java.util.concurrent 徹底解説 - momuki [[Link](https://omomuki-tech.com/archives/6369)]

- java.util.concurrentを試す - 技術メモ [[Link](https://tutuz-tech.hatenablog.com/entry/2019/05/04/093020)]

- JavaのThreadGroupを試してみる - 技術メモ [[Link](https://tutuz-tech.hatenablog.com/entry/2019/05/03/223352)]

- java.util.concurrentパッケージを用いたマルチスレッドプログラミングについて [[Link](http://www.02.246.ne.jp/~torutk/javahow2/concurrent.html)]



- Java Virtual Threadについて調べる時間 #coroutine - Qiita [[Link](https://qiita.com/rumblekat03/items/36f849c632597c3f6af7)]

- Virtual ThreadとJVM ThreadをSocketサーバで比較してみた | Atlas Developers Blog [[Link](https://devlog.atlas.jp/2023/03/29/5065)]

- Javaの仮想スレッドを学ぶ [[Link](https://zenn.dev/backpaper0/articles/b496359618180c)]

- Javaの新しいVirtual Threadを試してみた | 株式会社一創 [[Link](https://www.issoh.co.jp/tech/details/3209/)]

- Java21で正式リリースの Virtual Thread を使ってみた(その1) #Java - Qiita [[Link](https://qiita.com/miyo4/items/b4d39730269fa9a6f1fb)]

- Java21で正式リリースの Virtual Thread を使ってみた(その2) #JNI - Qiita [[Link](https://qiita.com/miyo4/items/e23cb97926ba220f1ef9)]

- Java仮想スレッドメモ(Hishidama's Virtual Threads Memo) [[Link](https://www.ne.jp/asahi/hishidama/home/tech/java/virtualthread.html)]





# Java その他

- Java標準ライブラリ javax.swing.text.html.parser の詳細解説 - momuki [[Link](https://omomuki-tech.com/archives/6414)]

- Java Stream API 完全ガイド：もうforループには戻れない！ - momuki [[Link](https://omomuki-tech.com/archives/6378)]
- java.nio.fileの強力な機能を使いこなす！モダンJavaファイルI/O完全ガイド - momuki [[Link](https://omomuki-tech.com/archives/6347)]





↓ 移動すべし

# GitLab

- Docker で GitLab と mailpit を使う:Linux 使い（略）Advent Calendar 2024 [https://zenn.dev/hiro345/articles/20241201_advent_calendar_2024_10]

- Docker 版 GitLab と mailpit の HTTPS 対応:Linux 使い（略）Advent Calendar 2024 [https://zenn.dev/hiro345/articles/20241201_advent_calendar_2024_16]

- GitLab CE初心者ガイド｜whale999 [[Link](https://note.com/whale999_/n/n452fe9385efa)]

- Gitセルフホスト最前線 — Forgejo vs Gitea vs GitLab、180コメントから見えた実態｜fit_gnu1832 [[Link](https://note.com/fit_gnu1832/n/n6d0fcc08ec84)]

  - GitLabは強力だが、ほとんどの場合オーバーキル。

  - GitLabは存在するだけで4〜8GB RAMを要求する。

  - Gitea/ForgejoにはGitHub Actions互換の「Actions」機能がある。
    ただし本家ほど成熟していないため、WoodpeckerというCI/CDを併用する人が多い。

  - rcloneでDockerボリュームを自動バックアップ ← これは何？

- CI/CD：誰にでも分かりやすい基本概念の解説｜Media Fusion [[Link](https://note.com/mediafusion_eng/n/n0655a209c649)]

- GitLab CI/CD 〜 Git と一体化した CI/CD の仕組み 〜 ｜Media Fusion [[Link](https://note.com/mediafusion_eng/n/na630aee54627)]

- Gitリポジトリをミラーリングする #GitHub - Qiita [[Link](https://qiita.com/oohira/items/175f68b4febd7b0342c0)]



- セルフホスト Git サービスをオススメする理由:Linux 使い（略）Advent Calendar 2024 [[Link](https://zenn.dev/hiro345/articles/20241201_advent_calendar_2024_01)]

- GitLabセルフホスティングの話 | ideaman's Notes [[Link](https://notes.ideamans.com/posts/2024/gitlab-selfhost.html)]

- Git サーバーをセルフホスティングしてみた [[Link](https://www.tojo.tokyo/posts/git-tojo-tokyo.html)]

- Gitのリモートリポジトリどこにするか #ホスティングサービス - Qiita [[Link](https://qiita.com/shu1rou/items/94548cbe1baaf591218b)]
  - GitBucket：インストール型、Scalaで実装、Javaサーブレットとして動作、GitLabより軽くて早いらしい。

- Gitea を推していく！ #gitea - Qiita [[Link](https://qiita.com/naaatsum3gu/items/74646af6794869350064)]
  - シングルバイナリで動作 (Go言語で実装)。業務使用は有償っぽい？

- GitBucket: A Git platform [[Link](https://gitbucket.github.io/)]

- GitBucket を使うことについて:Linux 使い（略）Advent Calendar 2024 [[Link](https://zenn.dev/hiro345/articles/20241201_advent_calendar_2024_12)]

- GitBucket の CI 用プラグインで CI/CD:Linux 使い（略）Advent Calendar 2024 [[Link](https://zenn.dev/hiro345/articles/20241201_advent_calendar_2024_13)]

- 10分ではじめるGitBucket #AWS - Qiita [[Link](https://qiita.com/msykiino/items/ce627e674b0352c50437)]

- たけぞう瀕死ブログ [[Link](https://takezoe.hatenablog.com/)] - 開発者のサイト

- GitHub, Bitbucket, GitLab, GitBucket について [[Link](https://zenn.dev/manase/scraps/4d2c3113b7cdb6)]

- GitBucketでGitHubリポジトリーのミラーを作成してみる - とことんDevOps | 日本仮想化技術のDevOps技術情報メディア [[Link](https://devops-blog.virtualtech.jp/entry/20230510/1683686692)]

- GitBucket に SSH アクセス用の公開鍵を登録する - Sig9 Memo v4.0 [[Link](https://sig9.org/blog/2024/01/12/)]

- 正直な話、gitは使いにくいと思いませんか？ - Quora [[Link](https://jp.quora.com/%E6%AD%A3%E7%9B%B4%E3%81%AA%E8%A9%B1-git%E3%81%AF%E4%BD%BF%E3%81%84%E3%81%AB%E3%81%8F%E3%81%84%E3%81%A8%E6%80%9D%E3%81%84%E3%81%BE%E3%81%9B%E3%82%93%E3%81%8B)]

  - GitHub - billiegoose/g: g - the improved CLI for git · GitHub [[Link](https://github.com/billiegoose/g)]
    例：

    | g                       | git equivalent                                               |
    | ----------------------- | ------------------------------------------------------------ |
    | g add *untracked_file*  | git add *untracked_file*                                     |
    | g rm *tracked_file*     | git rm -r *tracked_file* Then prompt y/n to delete untracked files in deleted directories. |
    | g stage                 | git add -u :/                                                |
    | g stage *tracked_file*  | git add *tracked_file*                                       |
    | g unstage               | git reset HEAD                                               |
    | g unstage *staged_file* | git reset HEAD *staged_file*                                 |
    | g reset                 | git checkout -f HEAD                                         |
    | g reset *file*          | git checkout *file*                                          |

  

  - 作業の場所とコマンドの関係図：
    ![作業の場所とコマンドの関係図](https://qph.cf2.quoracdn.net/main-qimg-49d7dbd747dbf802e08aac4cb792706a)

- Git x Eclipse : DevLife [[Link](https://devlife.blog.jp/archives/57451370.html)]



- GitBucketを利用したPull Request開発 | UNITRUST [[Link](https://www.unitrust.co.jp/1153)]
- [GitBucketで学ぶバージョン管理] #07 - プルリクエストしてみよう | なるーらぼ [[Link](https://nalu-labo.amebaownd.com/posts/594721/)]

- [GitBucketで学ぶバージョン管理] #08 - 共同でうまく改善していこう | なるーらぼ [[Link](https://nalu-labo.amebaownd.com/posts/604478/)]

- 大学のサークルでGitbucketとGitを使ってゲームコンテンツをチーム開発をしてみた #GitBucket - Qiita [[Link](https://qiita.com/kimarian/items/d95eda606eae13927145)]



- 混ぜるな危険 (msys2とCygwinとGit For Windowsを一緒に使ってはいけません) | OPCDiary [[Link](https://opcdiary.net/%E6%B7%B7%E3%81%9C%E3%82%8B%E3%81%AA%E5%8D%B1%E9%99%BA-msys2%E3%81%A8cygwin%E3%81%A8git-for-windows%E3%82%92%E4%B8%80%E7%B7%92%E3%81%AB%E4%BD%BF%E3%81%A3%E3%81%A6%E3%81%AF%E3%81%84%E3%81%91%E3%81%BE/)]

- Git BashとWSL2のプロンプトを揃えて気持ちよくする | 豆蔵デベロッパーサイト [[Link](https://developer.mamezou-tech.com/blogs/2023/09/10/gitbash-wsl2-prompt/)]
  - 実際に使う場合は WSL2+Git のみか、Git Bash のみかに絞った方が混乱しないのではないか？





# セミナー資料

- イベントレポート 第22回とことんDevOps勉強会「Gitで始めるバージョン管理入門」 - とことんDevOps | 日本仮想化技術のDevOps技術情報メディア [[Link](https://devops-blog.virtualtech.jp/entry/20240604/1717466567)]

- イベントレポート 第24回とことんDevOps勉強会「Docker互換のセキュアなコンテナ実行環境「Podman」超入門」 - とことんDevOps | 日本仮想化技術のDevOps技術情報メディア [[Link](https://devops-blog.virtualtech.jp/entry/20240725/1721892214)]

- イベントレポート 第21回とことんDevOps勉強会「今さら聞けないDocker入門 〜Dockerfileのベストプラクティス編」 - とことんDevOps | 日本仮想化技術のDevOps技術情報メディア [[Link](https://devops-blog.virtualtech.jp/entry/20240508/1715136606)]

- イベントレポート 第11回とことんDevOps勉強会「これから始めたい人集合！ゼロから学ぶGit/GitHub入門」 - とことんDevOps | 日本仮想化技術のDevOps技術情報メディア [[Link](https://devops-blog.virtualtech.jp/entry/20230428/1682649911)]







# Docker

- Dockerで本番運用！ログ、バッチ、バックアップ完全ガイド #Laravel - Qiita [[Link](https://qiita.com/t_sato_gradito/items/e5e30623cae354bd2b1d)]

- 事例から考えるDockerの本番利用に必要なこと | 進化を続けるDockerの今を知る | Think IT（シンクイット） [[Link](https://thinkit.co.jp/article/9701)] (古い 2016年)



- Dockerのデータをバックアップしたいのです（準備編） [[Link](https://blog.future.ad.jp/docker%E3%81%AE%E3%83%87%E3%83%BC%E3%82%BF%E3%82%92%E3%83%90%E3%83%83%E3%82%AF%E3%82%A2%E3%83%83%E3%83%97%E3%81%97%E3%81%9F%E3%81%84%E3%81%AE%E3%81%A7%E3%81%99%E6%BA%96%E5%82%99%E7%B7%A8)]

- Dockerのデータをバックアップしたいのです（実践編その1） [[Link](https://blog.future.ad.jp/docker%E3%81%AE%E3%83%87%E3%83%BC%E3%82%BF%E3%82%92%E3%83%90%E3%83%83%E3%82%AF%E3%82%A2%E3%83%83%E3%83%97%E3%81%97%E3%81%9F%E3%81%84%E3%81%AE%E3%81%A7%E3%81%99%E5%AE%9F%E8%B7%B5%E7%B7%A8%E3%81%9D%E3%81%AE1)]

- Dockerのデータをバックアップしたいのです（実践編その2） [[Link](https://blog.future.ad.jp/docker%E3%81%AE%E3%83%87%E3%83%BC%E3%82%BF%E3%82%92%E3%83%90%E3%83%83%E3%82%AF%E3%82%A2%E3%83%83%E3%83%97%E3%81%97%E3%81%9F%E3%81%84%E3%81%AE%E3%81%A7%E3%81%99%E5%AE%9F%E8%B7%B5%E7%B7%A8%E3%81%9D%E3%81%AE2)]

- Docker コンテナのバックアップ方法 [[Link](https://zenn.dev/yusuke_m/articles/dcfc7ee3c10082)]

- Dockerコンテナ内にあるファイルのバックアップ方法 #Dropbox - Qiita [[Link](https://qiita.com/The-town/items/758c487246a8ac89582f)]

- Docker コンテナのボリュームの内容をバックアップして復元する方法 – もばらぶエンジニアブログ [[Link](https://engineering.mobalab.net/2025/01/21/backup-and-restore-docker-container-volumes/)]

- 「コンテナ化」でより難しくなったアプリケーションのバックアップ、どうやって実現する？ | NetApp Blog [[Link](https://www.netapp.com/ja/blog/backing-up-container-operations/)]

- Dockerで小さく社内サービスを動かすTips - step by step building [[Link](https://hr-sano.net/blog/using-docker-my-tips/)]

- Dockerコンテナ↔Dockerイメージ↔バックアップファイル（バックアップ・リストア） #docker-image - Qiita [[Link](https://qiita.com/anlair/items/220ab8ad79a778bc1737)]





- お金をかけずにサーバーの勉強をしよう - プライベートの Dockerレジストリを作る - [[Link](https://subro.mokuren.ne.jp/0870.html)]

- 【Docker入門】 イメージ管理 [[Link](https://zenn.dev/cruway/articles/66fcf4a4ff78b0)]

- Dockerコンテナとコンテナイメージの管理 - Linux技術者認定 LinuC | LPI-Japan [[Link](https://linuc.org/study/column/4977/)]

- GitLabコンテナレジストリの管理 | GitLab Docs [[Link](https://docs.gitlab.com/ja-jp/administration/packages/container_registry/)]

- Docker プライベートレジストリを試す [[Link](https://zenn.dev/mnod/articles/31b201ad3bce98)]

  - CNCF Distribution [[Link](https://distribution.github.io/distribution/)]

  - HTTP API V2 | CNCF Distribution [[Link](https://distribution.github.io/distribution/spec/api/#deleting-an-image)] (Deleting an Image)

- [Docker] プライベートレジストリを構築して社内開発者にマスタイメージを配布できるようにする #Docker - Qiita [[Link](https://qiita.com/reiku19753/items/1abec75bf4c311bb61f5)]

- DockerのRegistryとPrivate registryについて | DevelopersIO [[Link](https://dev.classmethod.jp/articles/docker-registry-private-registry/)]

- Dockerで独自のプライベートローカルレジストリを使用する方法 | Docker Blog [[Link](https://www.docker.com/ja-jp/blog/how-to-use-your-own-registry/)] (古い:2013年)

- お金をかけずにサーバーの勉強をしよう - プライベートの Dockerレジストリを作る - [[Link](https://subro.mokuren.ne.jp/0870.html)]

- プライベートなDockerレジストリ構築手順 - にゃんざわ Think Memo Blog [[Link](https://think-memo.com/private-docker-registry/)]







... EoF ...
