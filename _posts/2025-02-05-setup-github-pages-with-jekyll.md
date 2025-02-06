---
layout: post
title:  "使用 Jekyll 建立 GitHub Pages"
date:   2025-02-05 16:05:00 +0800
categories: jekyll github-pages
---

## 緣起

在筆者的開發經驗中，偶爾但是會重複查詢一些不常使用的步驟或流程（例如：建立開發環境或不常修改的設定），並且發現許多開發者分享的內容較注重在技術與操作層面，較少提及選擇這些實作方式的原因與想法。因此，筆者希望透過技術部落格的方式，將自己的學習歷程與心得記錄下來，並且分享給大家。

## 選擇 GitHub Pages 與 Jekyll 的原因

1. 相較於使用他人建立的部落格平台，筆者更傾向於自建網站，將內容與服務的控制權掌握在自己手中，避免因為平台停止服務或變更政策而導致自己的心血付諸流水。

2. 若完全自行架設網站，需要自行建制伺服器與相關資安、網通設備，並需要使用對外的固定 IP、申請網域、 SSL 憑證等，會花費大量的時間與金錢。
   在考量上述的原因，並且預期網站的瞬間流量不大的狀況下，筆者選擇將自行架設的網站託管在 GitHub Pages 上，省去自行架站的成本，讓自己專注在內容的撰寫。要注意的是，[GitHub Pages](https://pages.github.com/) 只支援靜態網頁，不支援伺服器端語言（例如：PHP、Node.js），並且有網站大小、流量的使用限制，詳細的限制可以參考[官方文件](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#usage-limits)。

3. 為了避免自行撰寫網頁的額外開銷，並且希望內容的原始檔具備較高的可讀性，因此筆者希望可以透過 Markdown 語法快速的撰寫內容，並自動化產生網頁。因此需要一個將 Markdown 轉換成靜態網頁的工具。相關的工具有 [Jekyll](https://jekyllrb.com/)、[Hugo](https://gohugo.io/)、[Hexo](https://hexo.io/zh-tw/)，三者的比較可以參考[Jekyll vs Hexo vs Hugo](https://ithelp.ithome.com.tw/articles/10269253)。由於 GitHub Pages 已經整合 Jekyll，並且在網站建立初期，對網站編譯性能要求不大，因此筆者選擇使用 Jekyll 來建立自己的部落格，若未來對於性能要求較大，再轉換至 Hugo 或 Hexo。

## 安裝與設定[^1]

1. 在 GitHub 上建立一個新的 Public Repository，Repository Name 需為 `<username>.github.io`，其中 `<username>` 需與 GitHub 使用者名稱相同，例如：`ping-ee.github.io`。 GitHub Pages 會將該 Repository 的內容自動部署至 `https://<username>.github.io`。

2. 在本機安裝 Jekyll，可以參考[官方文件](https://jekyllrb.com/docs/installation/)，以下以 `Ubuntu 24.04` 為例說明安裝步驟[^2]。

   1. Jekyll 使用 Ruby 開發，因此需要先安裝 Ruby 與其他相依套件

      ```bash
      sudo apt-get update # 更新套件清單
      sudo apt-get install ruby-full build-essential zlib1g-dev
      ```

   2. Ruby 的套件 (Package) 稱作 `gem(s)`，並使用 RubyGems 作為Ruby的套件管理工具。為了將 `gems` 安裝在每個使用者的目錄下，而非 `root` 帳戶，因此設定 `GEM_HOME` 與 `GEM_PATH` 環境變數至 `~/.bashrc` 中，並將 `GEM_HOME/bin` 加入 `PATH` 中。

      ```bash
      echo "export GEM_HOME=\"\$HOME/.gem/ruby/$(ruby -e 'puts RUBY_VERSION')\"" >> ~/.bashrc
      echo "export GEM_PATH=\"\$GEM_HOME:\$GEM_PATH\"" >> ~/.bashrc
      echo "export PATH=\"\$GEM_HOME/bin:\$PATH\"" >> ~/.bashrc
      source ~/.bashrc # Apply the changes
      ```

   3. 安裝 Jekyll 與 Bundler 等套件

      Bundler 本身是一個 Gem，用途是 Ruby 的「專案」套件管理工具，可以讓 Ruby 使用正確的套件及其版本，避免相依性問題。

      ```bash
      gem install jekyll bundler
      ```

3. 在本機建立 Jekyll 專案
   1. 將 Repository Clone 至本機，並進入 Repository 目錄

      ```bash
      git clone <repository-url>
      cd <repository-name>
      ```

   2. 在 `<repository-name>/` 目錄下建立 Jekyll 專案

      ```bash
      jekyll new --skip-bundle .
      ```

      由於我們要在 GitHub Pages 上佈署網站，因此要調整專案使用到的 Gems，因此在建立專案時加上 `--skip-bundle`，先不安裝相依套件。

   3. 打開 `Gemfile` (位於專案根目錄下)，並做以下修改

       1. 移除或註解 `gem "jekyll"` 開頭的行
       2. 加入或取消註解 `gem "github-pages", group: :jekyll_plugins` 開頭的行，並參考 [GitHub Pages Dependency Versions](https://pages.github.com/versions/)，將該行修改為

            ```ruby
            gem "github-pages", "~> <github-pages-version>", group: :jekyll_plugins
            ```

   4. 將 `Gemfile` 存檔後，使用 `Bundler` 安裝相依套件

      ```bash
      bundle install
      ```

   5. Bundler 會將該專案的相依套件及其版本寫入 `Gemfile.lock` 中，我們可以將其加入 `.gitignore` 中

   6. 在 `_config.yml` 中設定網站的基本資訊，例如：`title`、`description`、`url` 等，如下所示

      ```yaml
      url: "https://<username>.github.io" # the base hostname and protocol for your site
      baseurl: "" # the subpath of your site, e.g. /blog
      ```

   7. 將 Repository Push 至 GitHub，GitHub Pages 會自動以 Jekyll 編譯並部署網站

      ```bash
      git add .
      git commit -m "Initial GitHub pages site with Jekyll"
      git push
      ```

## 在本機使用 Jekyll 預覽網站[^3]

```bash
bundle exec jekyll serve
```

根據輸出的訊息，打開瀏覽器並前往 `http://127.0.0.1:4000` 即可預覽網站。

## 參考資料

[^1]: [Create site with Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll)
[^2]: [Installing Jekyll on Ubuntu](https://jekyllrb.com/docs/installation/ubuntu/)
[^3]: [Test site locally with Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll)
