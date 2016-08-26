---
title: How to export twilights detected in GeolLight into FLightR
---

Following code should work:

There is `FLightR:::GeoLight2TAGS` function
it should work like this

```{r}
tags_data<-FLightR:::GeoLight2TAGS(raw, gl_twl) 
write.csv(tags_data, file='tmp.csv')
get_tags_data('tmp.csv')
```

BUT i would recommend use of [BAStag](https://github.com/SWotherspoon/BAStag) or [TwGeos](https://github.com/slisovski/TwGeos) instead of GeoLight for initial data processing
