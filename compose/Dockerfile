FROM ruby:2.3.1
MAINTAINER Marc Gascon <marc.gascon@gmail.com>

RUN gem install sinatra
RUN gem install redis
COPY ../hello-world.rb /app/hello-world.rb
WORKDIR /app/
ENV REDIS_HOST=redis

EXPOSE 4567
CMD ["ruby", "./hello-world.rb"]
