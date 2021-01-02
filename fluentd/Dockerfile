# fluentd/Dockerfile
#FROM fluent/fluentd
#RUN ["gem", "install", "fluent-plugin-elasticsearch", "--no-rdoc", "--no-ri", "--version", "1.9.7"]

FROM fluent/fluentd:v1.9.1-debian-1.0
User root
RUN gem install fluent-plugin-elasticsearch
User fluent
