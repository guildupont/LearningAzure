FROM php:7.4-apache
RUN rm -rvf /var/www/html/*
RUN echo '<html><head></head><body>This is OS host <?php echo exec("uname -n 2>&1"); ?><br><br><br>Metadata: <?php echo exec("curl http://169.254.169.254/metadata/instance?api-version=2021-01-01 -H Metadata:true 2>&1")?> </body>' > /var/www/html/index.php
