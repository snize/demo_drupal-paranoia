## Drupalをインストールして動作確認

```
drush site:install demo_umami --db-url="sqlite://sites/default/files/.ht.sqlite" -y
cd web && php -S localhost:8888
```