FROM nginx:alpine

RUN rm -rf /usr/share/nginx/html/*

COPY app/Trend/dist/ /usr/share/nginx/html/

RUN ls -l /usr/share/nginx/html/

EXPOSE 3000

CMD ["nginx", "-g", "daemon off;"]
