> if I wanted to know *everything* that was going to happen in the next X cycles. Can ask ‘what is going to happen at cycle X?’

ok say you have a pattern `let mypattern = (every 2 (fast 2) "a b c") :: Pattern String`. There's a function `arc` for getting events out of it. An arc is basically a timespan. So to get the first cycle you'd do `arc mypattern (0,1)` and get `[((0 % 1,1 % 6),(0 % 1,1 % 6),"a"),((1 % 6,1 % 3),(1 % 6,1 % 3),"b"),((1 % 3,1 % 2),(1 % 3,1 % 2),"c"),((1 % 2,2 % 3),(1 % 2,2 % 3),"a"),((2 % 3,5 % 6),(2 % 3,5 % 6),"b"),((5 % 6,1 % 1),(5 % 6,1 % 1),"c")]`. Each event has two arcs, for the purposes of visualisation you can just take the second one I think. That's just a pattern of strings. A parameter is a pattern of associative arrays



> If i was trying to get events from `d3 $ every 2 (fast 2) $ sound "bd [hh, lt]" # gain "0.2" # orbit "0"`
> would i need to separately define it as a pattern using `let`?

You could do `arc (every 2 (fast 2) $ sound "bd [hh, lt]" # gain "0.2" # orbit "0") (0,4)` to get the first four cycles worth for example. In truth that `arc` function is all a pattern is




[2:58] 
instead of `(0,4)`, could I do like `(currentcycle, currentcycle+x)` ..?


[2:59] 
would that just be `(now, now+x)`?


[3:01] 
this is what I would like to happen ideally:


[3:01] 
every time I run or update and run a pattern e.g. `d1 $ every 2 (fast 2) $ sound "bd"`


[3:02] 
simultaneously it’s next 16 cycles worth of events should be emitted over OSC


[3:05] 
 ```do
  let p = every 2 (fast 2) $ sound "bd [hh, lt]" # gain "0.2" # orbit "0"
      lookahead = 4
  d1 $ p
  arc (p) (now,now+lookahead)
```
```<interactive>:995:3: error:
    • Couldn't match type '[]' with 'IO'
      Expected type: IO (Event ParamMap)
        Actual type: [Event ParamMap]
    • In a stmt of a 'do' block: arc (p) (now, now + lookahead)
      In the expression:
        do { let p = every 2 (fast 2)
                     $ sound "bd [hh, lt]" # gain "0.2" # orbit "0"
                 lookahead = 4;
             d1 $ p;
             arc (p) (now, now + lookahead) }
      In an equation for 'it':
          it
            = do { let p = ...
                       ....;
                   d1 $ p;
                   arc (p) (now, now + lookahead) }
```


yaxu [3:06 PM] 
```import Sound.OSC.FD

let wrapDirts ds = do x <- openUDP "127.0.0.1" 3000
                      let f d p = do now <- getNow
                                     sendOSC x $ Message "/vis" [string $ show $ arc p (now,now+16)]
                                     d p
                          fs = map f ds
                      return fs
```



[3:07] 
then `[x1,x2,x3,x4,x5] <- wrapDirts [d1,d2,d3,d4,d5]`


jarm [3:10 PM] 
that is brilliant


yaxu [3:10 PM] 
maybe you want to identify which stream is being updated tho?


jarm [3:10 PM] 
yeah that was a previous question


[3:11] 
is that possible?


yaxu [3:12 PM] 
```let wrapDirts ds = do x <- openUDP "127.0.0.1" 3000
                      let f (n,d) p = do now <- getNow
                                         sendOSC x $ Message "/vis" [string $ show $ arc p (now,now+16), int32 n]
                                         d p
                          fs = map f (enumerate ds)
                      return fs
```


[3:13] 
that needs `import Sound.Tidal.Utils`
