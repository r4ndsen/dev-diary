# dev diary

The lazy developer's diary.

### usage
```shell
// open today's diary entry
$ diary

// open tomorrow's diary entry
$ diary 1
```

Add this script to your `~/.bashrc` or `~/.zshrc`

```shell
function diary() {
    // setup
    local diaryRepoPath=~/dev-diary

    // dynamically pick your editor
    local editor=$EDITOR
    if [[ -z "$editor" ]]; then
        editor=open
    fi

    // or hard pick it
    editor=code

    local timeAdjust=""

    if [ -n "$1" ]; then
       timeAdjust="-v+$1d"
    fi

    local weekDay=$(date $timeAdjust +'%u')
    if [[ $weekDay -gt 5 ]]; then
       echo "it's not a workday, shush!"
       return
    fi

    local folderPath="$diaryRepoPath/$(date $timeAdjust +'%Y/%m - %B')"
    mkdir -p $folderPath

    # cast "01" to "1"
    local day=$(expr $(date $timeAdjust +'%d') + 0)

    local daySuffix="th"
    if [[ $day -eq 1 || $day -eq 21 || $day -eq 31 ]]; then
        daySuffix="st"
    elif [[ $day -eq 2 || $day -eq 22 ]]; then
        daySuffix="nd"
    elif [[ $day -eq 3 || $day -eq 23 ]]; then
        daySuffix="rd"
    fi

    local dayFile="$folderPath/$(date $timeAdjust +'%d').md"
    if [ ! -f $dayFile ]
    then
        echo "# $(date $timeAdjust +"%A, %B $day$daySuffix %Y")" >> $dayFile \
        && cat "$diaryRepoPath/template.md" >> $dayFile \
        && git -C "$diaryRepoPath" add "$diaryRepoPath/." \
        && git -C "$diaryRepoPath" commit -m "add diary for $(date $timeAdjust +'%F')" \
        && git -C "$diaryRepoPath" push origin main
    fi

    $editor "$diaryRepoPath"

    local yesterdayFile="$diaryRepoPath/$(date -v-1d +'%Y/%m - %B')/$(date -v-1d +'%d').md"
    if [[ -f "$yesterdayFile" && ! -s "$yesterdayFile" ]]; then
        $editor "$yesterdayFile"
    fi

    $editor "$dayFile"
}
```
