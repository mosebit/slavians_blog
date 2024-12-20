---
layout: post
title:  "Как создать сайт при помощи Jekyll на базе "
date:   2024-11-30 13:43:57 +0300
categories: jekyll update
---

**Дисклеймер:**

Я не являюсь веб-разработчиком и не тратил много времени на поиск лучшего генератора статических сайтов (прочитал несколько статей и выбирал между Hugo и Jekyll, в итоге выбор пал на второй вариант, так как в Jekyll встроена поддержка GitHub Pages и в самой документации GitHub Pages описывается работа с Jekyll).

Данная статья преследует одну цель - рассказать как создавался данный блог.

----

### Коротко о Jekyll и GitHub Pages

Jekyll является генератором статических сайтов, что означает возможность при его помощи сгенерировать сайт из заранее заготовленного контента на каком-либо языке разметки (например, Markdown) и определенных тем.

Самостоятельно разрабатывать темы не требуется - в репозитории Jekyll можно найти заготовки на любой вкус и цвет:

- https://github.com/topics/jekyll-theme
- https://jamstackthemes.dev/ssg/jekyll/
- http://jekyllthemes.org

Немного напрячься предстоит один раз - на этапе подготовке системы, а далее все новые страницы и посты будут добавляться за считанные секунды (даже с учетом сборки сайта).

**GitHub Pages** — это служба хостинга статических сайтов, которая принимает файлы HTML, CSS и JavaScript прямо из репозитория на GitHub, при необходимости запускает файлы в процессе сборки и публикует веб-сайт.

### Пора установить этот Джекуль

В повседневной жизни я использую Windows, поэтому было приятно увидеть данную ОСь в списке гайдов по установке на [соответствующей странице документации](https://jekyllrb.com/docs/installation/). "Провалившись" в этот гайд на самом верху можно увидеть:

"While **Windows is not an officially-supported platform**, it can be used to run Jekyll with the proper tweaks." - https://jekyllrb.com/docs/installation/windows/

Мой жизненный опыт (для предотвращения ухудшения нервной системы) советует по возможности избегать работу с чем-то по типу *"not an officially-supported platform"*.

К счастью, пользователи Windows могут воспользоваться WSL. *Про данную подсистему Windows у меня уже есть статья - [Что такое WSL]({% post_url 2024-12-12-What-is-WSL %})*

В WSL я использую дистрибутив Ubuntu Linux и действую по [инструкции](https://jekyllrb.com/docs/installation/ubuntu/):

#### Установка Ruby и других необходимых компонентов (рекомендуется устанавливать не от пользователя root): 
``` sh
sudo apt-get install ruby-full build-essential zlib1g-dev
```

Перед установкой необходимо настроить `.bashrc`:
``` sh
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

<details>

<summary>Что такое .bashrc</summary>

Можно начать с имени самого файла и первое, что может удивить начинающего пользователя Linux это точка в начале имени файла. В контексте Linux файлы с таким именованием чаще всего называют "скрытыми" или "hidden". Скрыты данные файлы только от команды `ls` и шаблонов подстановки имен файлов, а возможность их чтения/изменения/исполнения зависят только от настроенных прав доступа.



`.bashrc` — это файл конфигурации для среды оболочки Bash. Каждый раз, когда запускается интерактивный сеанс оболочки Bash, выполняется файл сценария .bashrc. Файл содержит различные комментарии, конфигурации и функции для настройки работы оболочки и автоматизации задач.

</details>


И теперь пора установить сами пакеты `Jekyll` и `Bundler`:

```
gem install jekyll bundler
```

#### Создание репозитория для сайта

На GitHub создается репозиторий, клонируется на хост, с которого будет вестись разработка сайта, все это описано на [соответствующей странице](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll) документации GitHub Pages.

Помимо прочего на вышеуказанной странице описаны шаги по созданию заготовки сайта Jekyll: 

```
To create a new Jekyll site, use the jekyll new command in your repository's root directory:

$ jekyll new --skip-bundle .
# Creates a Jekyll site in the current directory
.....
```

Не буду дублировать источник, ведь в нем все описано предельно ясно.

### Как добавить темную тему на Jekyll-based сайт

Сайт запущен, но не сразу устраивает, что нет переключения между темами оформления.

Основу логики смены тем взял из инструкции в [блоге scottesbrandt](https://www.scottesbrandt.com/jekyll/2024/04/25/Enable-dark-mode-on-jekyll-site.html). Но с данной темой есть небольшая проблема - при выборе темной темы при попытке обновления страницы на момент загружается светлая тема и только после включается темная.

Данная проблема не новая и чаще всего называется "theme flashing".

Для ее решения пришлось переопределить файл `assets/main.scss`. Подробнее ознакомиться с переопределением настроек тем по умолчанию можно [здесь](https://jekyllrb.com/docs/themes/#overriding-theme-defaults), файл `assets/main.scss` после правок:

```
---
# Only the main Sass file needs front matter (the dashes are enough)
---

@import "minima";

html {
    opacity: 0;
    transition: opacity 1s;
}

html.light, html.dark {
    opacity: 1;
}
```

Скрипт смены темы (который изначально взял из блога [scottesbrandt](https://www.scottesbrandt.com/jekyll/2024/04/25/Enable-dark-mode-on-jekyll-site.html)) тоже пришлось изменить:

```
// Setup darkreader for CORS
DarkReader.setFetchMethod(url => {
    let headers = new Headers()
    headers.append('Access-Control-Allow-Origin', '*')
  
    return window.fetch(url, {
      headers,
      mode: 'no-cors',
    })
  })
  
  function darkModeSelected() {
    let darkModeSelected = localStorage.getItem('darkMode');
  
    if (darkModeSelected === "false"){
      return false;
    }
    if (darkModeSelected === "true") {
      return true;
    }
    return darkModeSelected;
  }
  
  function darkModeEnabled() {
    if (darkModeSelected() === true) {
      return true;
    }
    if (darkModeSelected() === null && darkModePreferred === true) {
      return true;
    }
    return false;
  }
  
  // set color mode and icons on page load
  function checkDarkMode() {
    if (darkModeEnabled()) {
      DarkReader.enable();
      document.documentElement.classList.remove('light');
      document.documentElement.classList.add('dark');
  } else {
      DarkReader.disable();
      document.documentElement.classList.remove('dark');
      document.documentElement.classList.add('light');
  }
  
    if (darkModeSelected() === true) {
      setIconsDarkModeOn();
    } else if (darkModeSelected() === false) {
      setIconsDarkModeOff();
    }
  }
  
  function toggleDarkMode() {
    if (darkModeEnabled()) {
      disableDarkMode();
    } else {
      enableDarkMode();
    }
  }
  
  function enableDarkMode() {
    DarkReader.enable();
    localStorage.setItem('darkMode', 'true');

    document.documentElement.classList.remove('light');
    document.documentElement.classList.add('dark');

    setIconsDarkModeOn();
  }
  
  function disableDarkMode() {
    DarkReader.disable();
    localStorage.setItem('darkMode', 'false');

    document.documentElement.classList.remove('dark');
    document.documentElement.classList.add('light');

    setIconsDarkModeOff();
  }
  
  function setIconsDarkModeOn() {
    iconLight.style.color = "grey";
    iconDark.style.color = "blue";
  }
  
  function setIconsDarkModeOff() {
    iconDark.style.color = "grey";
    iconLight.style.color = "blue";
  }
```

Возможно это не самое элегантное решение, но оно работает.

FYI: Сайт не находится в своем финальном состоянии и с течением времени будет неоднократно обновляться, следите, если интересно!