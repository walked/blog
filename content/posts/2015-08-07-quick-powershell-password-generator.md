---
title: "Quick Tidbit: PowerShell Random Password Generator"
date: 2015-08-07T12:32:30-05:00
draft: false
tags: ["powershell"]
---


I had a need for a quick and dirty password generator for an AD user creation script. There's likely a better way to do this, but I wanted to have it fairly modular so it can be customized on the fly to meet organizational password requirements.

<!--more-->
```powershell
$PW_LENGTH = 8
$SPEC_CHAR = 1
$CAP_LETTER = 1
$DIGITS = 1

function Create-Password{

    #Create the different classes of password characters
    $chars = 'abcdefghijkmnopqrstuvwxyz'
    $caps = 'ABCEFGHJKLMNPQRSTUVWXYZ'
    $nums = '23456789'
    $special = '!#%&'
    $all = $chars + $caps + $nums + $special

    #Instantiate a password with an empty string (note the space) so we can insert into it
    $pwd = ' '

    <# 
    #SPECIAL CHARACTERS
    #Insert special characters for the specified minimum
    #>
    for($i =0; $i -lt $SPEC_CHAR; $i++)
    {
        $pwd = $pwd.insert((get-random -min 0 -max $pwd.length), $special[(Get-Random -min 0 -max ($special.Length))])
    }

    <# 
    #CAPITAL LETTERS
    #Insert capital characters for the specified minimum
    #>
    for($i =0; $i -lt $CAP_LETTER; $i++)
    {
        $pwd = $pwd.insert((get-random -min 0 -max $pwd.length), $caps[(Get-Random -min 0 -max ($caps.Length))])
    }

    <# 
    #NUMERICAL DIGITS
    #Insert digits for the specified minimum
    #>
    for($i =0; $i -lt $DIGITS; $i++)
    {
        $pwd = $pwd.insert((get-random -min 0 -max $pwd.length), $nums[(Get-Random -min 0 -max ($nums.Length))])
    }

    <#
    FINISH UP
    Insert characters from all available sets until we meet the minimum password length
    #>
    do{
        $pwd = $pwd.insert((get-random -min 0 -max $pwd.length), $all[(Get-Random -min 0 -max ($all.Length))])
    }while(($pwd.Trim()).length -lt $PW_LENGTH)

    #Get rid of whitespace
    $pwd = $pwd.Trim()

    return $pwd

}
```