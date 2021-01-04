FROM ruby:2.5.5-alpine3.9 as builder

RUN apk add --no-cache --update build-base \
    linux-headers \
    sqlite \
    sqlite-dev \
    postgresql-dev \
    nodejs \ 
    yarn \
    tzdata

WORKDIR /app

ENV RAILS_ENV=production
ENV NODE_ENV=production
ENV BUNDLER_VERSION=2.2.3

COPY Gemfile Gemfile.lock /app/
RUN gem install bundler -v 2.2.3 
RUN gem update --system
RUN gem uninstall bundler -v 1.17.3
RUN bundle config set --local deployment 'true' 
RUN bundle config set --local without 'development:test'
RUN bundle config path 'vendor/bundle' --local
RUN bundle install --jobs 4
RUN yarn install --ignore-engines
COPY . /app/

RUN bin/rails assets:precompile
############################################################
FROM ruby:2.5.5-alpine3.9

RUN apk add --no-cache --update postgresql-dev \
    sqlite-dev \
    linux-headers \
    sqlite \
    nodejs \ 
    tzdata

ENV RAILS_ENV=production

RUN gem install bundler -v 2.2.3 
RUN gem update --system
RUN gem uninstall bundler -v 1.17.3

WORKDIR /app
COPY . /app/

COPY --from=builder /usr/local/bundle/config /usr/local/bundle/config
COPY --from=builder /app/vendor/bundle/ /app/vendor/bundle/
COPY --from=builder /app/public/ /app/public/
CMD ["bin/rails", "server"]
