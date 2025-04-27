- what are the predefined traveler types ?
- in the prompt hotel category will be different for all the location ? in prompt or white editing
- what will be the vehicles type ? send the values.
- their will be only one vehicle for the package or can be multiple ? in prompt or white editing
- hotel type will be added from BE [1start, 2start ... 5 start] ?
-

stars , vehicles only one ,

server_name api-dev.tripinnov.com;

server {
listen 80;
server_name https://quiksale.in;

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

}

ln -s /etc/nginx/sites-available/quiksale /etc/nginx/sites-enabled/

nginx -t
systemctl restart nginx
