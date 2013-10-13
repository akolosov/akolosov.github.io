---
layout: post
title:  "Настраивай это"
date:   2013-10-13 10:00:00
tags:
- ruby
- rails
- gem
---
Когда у любого проекта появляются пользователи, это приносит первые радости и пержые же головные боли. Одному нужно то, второму чтобы тоже самое, но не как у всех и прочие "прелести". Именно тут помогают индивидуальные настройки пользователей. Разумеется они не решают всех вышеописанных проблем, но очень помогают в их решении. Тут главное не переусерсотвовать с количеством пользовательских настроек и не запутаться самому. Хочу расказать про свой опыт в этом вопросе. Я расскажу про gem [store_configurable](https://github.com/metaskills/store_configurable). Это очень простой и понятный в использовании gem. 

Для установки выполните команду:

{% highlight ruby %}

    gem install store_configurable

{% endhighlight %}

... и добавьте в Gemfile строку:

{% highlight ruby %}

    gem 'store_configurable'

{% endhighlight %}

Далее создайте миграцию для создания в модели пользователя поля, где будет храниться собственно сама его настройка:

{% highlight ruby %}

    rails g migration AddConfigToUsers _config:text
    ...
    rake db:migrate

{% endhighlight %}

Добавляем в модель User или ту, что у вас вполняет эти функции в начало файла строку store_configurable:

{% highlight ruby %}

    class User < ActiveRecord::Base
        store_configurable
        
        # бла-бла-бла
        
    end

{% endhighlight %}

Так же неплохо добавить в модель метод для настройки конфигурации по умолчанию для вновь созданных пользователей или для тех у кого еще нет настроек:

{% highlight ruby %}

    before_save :default_config

    protected

    def default_config
        self.config.refresh_interval = 180 if self.config.refresh_interval.empty?
        self.config.time_zone = 'Moscow' if self.config.time_zone.empty?
    end

{% endhighlight %}

В контроллере создаем методы для редактирования:


{% highlight ruby %}

    # GET /users/1/setup
    def setup
        @user = User.find(params[:id])
    end

{% endhighlight %}

... сохранения настроек:

{% highlight ruby %}

    # PUT /users/1/save_settings
    def save_settings
        @user = User.find(params[:id])

        respond_to do |format|
            params[:user][:config].each do |name, value|
                @user.config[name] = value
            end

            if @user.save
                format.html { redirect_to :root, notice: 'Настройки пользователя успешно обновлены.' }
            else
                format.html { redirect_to :user_setup, alert: 'Неверные параметры настройки! Попробуйте снова!' }
            end
        end
    end

{% endhighlight %}


... и форму для их редактирования:

{% highlight haml %}

    = simple_form_for @user, url: user_settings_path(@user), method: :put, html: { class: 'form-horizontal form', id: 'form' } do |f|
      .form-header
        %h3= "Изменение настроек пользователя "+@user.realname
      .form-inputs
        = f.simple_fields_for :config do |c|
          .control-group.string.user_config_timezone
            %label.select.control-label Временная зона
            .controls
              = c.time_zone_select :time_zone, ActiveSupport::TimeZone.all.sort, model: ActiveSupport::TimeZone, default: @user.config.time_zone, class: 'form-input'
              = c.input :refresh_interval, label: 'Время обновления списка, секунд', required: false, input_html: { class: 'form-input', value: @user.config.refresh_interval }
      .form-actions
        .pull-right.btn-group
          = f.submit 'Сохранить настройки', class: 'btn btn-success'
          = link_to 'Отмена', :root, class: 'btn

{% endhighlight %}

Как-то так... Я же говорил, что все будет просто и понятно.
