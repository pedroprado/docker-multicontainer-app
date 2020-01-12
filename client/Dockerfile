#fase 1: fase de build
FROM node:alpine as builder
WORKDIR '/app'
COPY package.json ./
RUN npm install
COPY ./ ./
RUN npm run build

#fase 2: fase de run (usando um segundo FROM, o docker automaticamente descarta a imagem anterior)
FROM nginx
#expose é necessário para o Elasticbeanstalk. Ele faz o mapeanto do container a partir da porta definida no expose
EXPOSE 3000
#colocar as configurações customizadas
COPY ./default.conf /etc/nginx/conf.d/default.conf
#aqui estamos copiando tudo o que foi produzido na fase de build (arquivos para produção) para se disponibilizado pelo ngnix em produção    
COPY --from=builder /app/build /usr/share/ngnix/html