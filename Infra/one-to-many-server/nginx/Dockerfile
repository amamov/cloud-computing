FROM nginx:latest

COPY nginx.conf /etc/nginx/nginx.conf
COPY nginx-web1.conf /etc/nginx/sites-available/
COPY nginx-web2.conf /etc/nginx/sites-available/

RUN mkdir -p /etc/nginx/sites-enabled/
RUN ln -s /etc/nginx/sites-available/*.conf /etc/nginx/sites-enabled/

#EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
