# terminal colors

## Colors and character attributes
These are actually terminal capabilities and not related to shell. But since I use these in my bash scripts, putting them here seems reasonable.

todo - discuss:
setaf/setab vs setf/setb


Change the foreground and background colors with tput
```
# play with the foreground colors
for ((i=0;i<8;i++))
do
    tput setaf $i
    echo "tput setaf $i produces this color"
    tput sgr0 # restore default
done

 # play with the background colors
for ((i=0;i<8;i++))
do
    tput setab $i
    echo "tput setab $i produces this color"
    tput sgr0 # restore default
done
```
or for finer control, embed the escape sequences right into your string (tput is just a short cut to these escape values anyways)
```
# play with the foreground colors
for ((i=30;i<38;i++))
do
    echo -e "\e[0${i}m escape seq $i produces this color\e[0m"
done

# play with the background colors
for ((i=40;i<48;i++))
do
    echo -e "\e[0${i}m escape seq $i produces this color\e[0m"
done

# or do them together
for ((i=30;i<38;i++))
do
    for ((j=40;j<48;j++))
    do
        echo -ne "\e[0${i};${j}m $i:$j \e[0m"
    done
    echo
done
```

Other display attributes

| attribute | escape | tput enable | tput disable |
| ---|---|---|---|
| none|\e\[0m|sgr0|n/a |
| bold|\e\[1m|bold|sgr0 |
| dim|\e\[2m|n/a|n/a |
| emphasis|\e\[3m|n/a|sgr0 |
| underline|\e\[4m|smul|rmulc |
| blink|\e\[5m|blink|sgr0 |
| reverse|\e\[7m|rev|sgr0  |
| invisible|\e\[8m|invis|sgr0 | 
| strikethrough| \e\[9m | n/a | n/a |


(revisit these: terminfo has evolved since I first made these notes)

Simple attribute demo:
```
for ((i=0;i<10;i++))
do
    echo -e "\e[${i}mshow me what '\\\e[${i}m' does\e[0m"
done
```
