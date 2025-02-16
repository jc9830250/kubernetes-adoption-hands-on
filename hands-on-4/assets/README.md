# 作業架構說明
## Nginx設定
   * Nginx configMap: Nginx-conf.yaml  
     Client導到 Wordpress Service  
   * Nginx deployment: nginx-deployment.yaml
     * nginx-deployment 
       volumeMounts:   
       host-log-volume 與fluented共用log Path 使用Host Path  
       nginx-conf-volume 使用 Nginx configMap  
     * nginx-service  
     使用NodePort對外  
   * Nginx HPA: nginx-hpa.yaml  
     minReplicas: 2  
     maxReplicas: 5  
     targetCPUUtilizationPercentage: 70% CPU 70%後進行拓展  
## fluented設定
 * fluentd configMap: fluentd-conf.yaml  
    fluent.conf抓取Nginx log並輸出
 * fluentd daemonset: fluentd-daemonset.yaml     
    * fluentd-daemonset  
      volumeMounts:  
      fluentd-conf-volume 使用fluent congfigMap  
      host-log-volume 與Nginx共用log Path 使用Host Path  
 ## Wordpress設定
  * Mysql Secrect: mysql-secrect.yaml  
    mysql db資料
  * wordpress-deployment: wordpress-deployment.yaml  
    * wordpress-deployment  
       * env 使用 mysql secrect的 mysql-secret
         name: WORDPRESS_DB_HOST  
          value: "mysql-service:3306" mysql service  
        * name: WORDPRESS_DB_USER: mysql_wordpress_user  
        * name: WORDPRESS_DB_PASSWORD: mysql_wordpress_password  
        * name: WORDPRESS_DB_NAME: mysql_db_name  
  * wordpress-service    
  type: ClusterIP
  * wordpress-hpa : wordpress-hpa.yaml  
    minReplicas: 2  
    maxReplicas: 5  
    cpu Utilization  30
  ## Mysql設定
  * Mysql Secrect: mysql-secrect.yaml  
    mysql db資料
  * mysql-statefulset: mysql-statefulset.yaml  
    * mysql-statefulset
    * env 使用 mysql secrect的 mysql-secret
        * name: MYSQL_ROOT_PASSWORD: mysql_root_user  
        * name: MYSQL_DATABASE: mysql_wordpress_password  
        * name: MYSQL_PASSWORD: mysql_db_name  
    * volumeMounts
      * storageClassName: mysql-pv  
      * accessModes: [ "ReadWriteOnce" ]  
      * storage: 1Gi  
  * mysql-service  
  clusterIP: None
  * mysql-pv-volume  
    * storageClassName: mysql-pv
    * storage: 1Gi
    * accessModes: ReadWriteOnce
    * hostPath: path: "/run/desktop/mnt/host/C/Users/jc098/Desktop/data"
 * mysql-pv-claim  
   PersistentVolumeClaim  
   storageClassName: mysql-pv  
