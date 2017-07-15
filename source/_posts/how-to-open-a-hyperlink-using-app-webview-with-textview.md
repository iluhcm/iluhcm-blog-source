title: 使用Android内置WebView打开TextView中的超链接
date: 2016-07-06 10:27:57
tags: [Android, WebView, TextView, HyperLink, SpannableStringBuilder, Html]
categories: Android
---

## 问题缘由

**产品说，我们要实现问答功能，答案内的链接要使用内置的浏览器打开。

**视觉说，我们要给超链接标上我们自己的颜色。

如图：

![功能需求](http://7xl6ic.com1.z0.glb.clouddn.com/blog_textview_clickable_hyperlink.jpg)

下面我们分析下如何实现。

## 使用Html

常规方法，给定一段标准html文档，使用Html.fromHtml()封装，直接使用TextView显示。

```
TextView textView = (TextView) findViewById(R.id.detailed_question_tv_answer);
String testString =
"亲,一般遇到这问题您可以这样哦:<br>1.可以<font color='#ff8785'><a href='http://m.kaola.com'>催发货</a></font>哦~<br>2.然后耐心等待哦~<br>3.1-3天后新也可以拨打我们的客服.";
textView.setMovementMethod(LinkMovementMethod.getInstance());
// 设置链接颜色
textView.setLinkTextColor(getResources().getColor(R.color.red_ff8785));
Spanned htmlString = Html.fromHtml(testString);
textView.setText(htmlString);
```

使用常规方法无论怎么设置，链接都会使用隐式Intent打开，即使用外部的浏览器打开，不符合咱们产品的需求呀。怎么才能监听这个使用并使用内部WebView打开呢？使用SpannableStringBuilder。

## 使用SpannableStringBuilder

直接上代码。

```
TextView textView = (TextView) findViewById(R.id.detailed_question_tv_answer);
String testString =
    "亲,一般遇到这问题您可以这样哦:<br>1.可以<font color='#ff8785'><a href='http://m.kaola.com'>催发货</a></font>哦~<br>2.然后耐心等待哦~<br>3.1-3天后新也可以拨打我们的客服.";
textView.setMovementMethod(LinkMovementMethod.getInstance());
textView.setLinkTextColor(getResources().getColor(R.color.red_ff8785));
String linkText = "催发货";
int startIndexOfLink = testString.indexOf(linkText);
int endIndexOfLink = startIndexOfLink + linkText.length();
SpannableStringBuilder spannableStringBuilder = new SpannableStringBuilder(testString);
spannableStringBuilder.setSpan(new ClickableSpan() {
    @Override
    public void onClick(View widget) {
        ActivityUtils.startWebviewActivity(DetailedQuestionActivity.this, "http://m.kaola.com", false);
    }
}, startIndexOfLink, endIndexOfLink, Spannable.SPAN_INCLUSIVE_EXCLUSIVE);
textView.setText(spannableStringBuilder);
```

当然，这个方法是有很大的局限性的，必须知道链接在文案中的具体位置，以及链接的地址才能够使用这种方法。按照这种思路，我们必须使用正则表达式获取对应的a标签才能得到链接。这种方法拿到的链接在文案中的具体位置是难以把握的，很有可能出错。

## Html + SpannableStringBuilder

有没有第三种方法，即能够解析到给定文案中的所有Html标签，又能够使用内置的WebView打开这个链接？从第一种方法中，我们直接使用`Html.fromHtml()`方法拿到对应的Spanned结果，我们可以从这里入手，看看这个方法是怎么解析html标签的

```
public static Spanned fromHtml(String source, ImageGetter imageGetter,
                               TagHandler tagHandler) {
    // 使用org.ccil.cowan.tagsoup.Parser作为解析器                           
    Parser parser = new Parser();
    try {
        parser.setProperty(Parser.schemaProperty, HtmlParser.schema);
    } catch (org.xml.sax.SAXNotRecognizedException e) {
        // Should not happen.
        throw new RuntimeException(e);
    } catch (org.xml.sax.SAXNotSupportedException e) {
        // Should not happen.
        throw new RuntimeException(e);
    }
    // 使用HtmlToSpannedConverter将Ttml转换成Spanned
    HtmlToSpannedConverter converter =
            new HtmlToSpannedConverter(source, imageGetter, tagHandler,
                    parser);
    return converter.convert();
}
```

接下来看一下`HtmlToSpannedConverter.convert()`这个方法。`HtmlToSpannedConverter`实现了`ContentHandler`接口，`ContentHandler`用于处理Xml文档的解析细节。

```
public Spanned convert() {

    mReader.setContentHandler(this);
    try {
        mReader.parse(new InputSource(new StringReader(mSource)));
    } catch (IOException e) {
        // We are reading from a string. There should not be IO problems.
        throw new RuntimeException(e);
    } catch (SAXException e) {
        // TagSoup doesn't throw parse exceptions.
        throw new RuntimeException(e);
    }

    // Fix flags and range for paragraph-type markup.
    Object[] obj = mSpannableStringBuilder.getSpans(0, mSpannableStringBuilder.length(), ParagraphStyle.class);
    for (int i = 0; i < obj.length; i++) {
        int start = mSpannableStringBuilder.getSpanStart(obj[i]);
        int end = mSpannableStringBuilder.getSpanEnd(obj[i]);

        // If the last line of the range is blank, back off by one.
        if (end - 2 >= 0) {
            if (mSpannableStringBuilder.charAt(end - 1) == '\n' &&
                mSpannableStringBuilder.charAt(end - 2) == '\n') {
                end--;
            }
        }

        if (end == start) {
            mSpannableStringBuilder.removeSpan(obj[i]);
        } else {
            mSpannableStringBuilder.setSpan(obj[i], start, end, Spannable.SPAN_PARAGRAPH);
        }
    }

    return mSpannableStringBuilder;
}
```

我们关心Html是如何被转换成Spanned就够了。在整个解析Html的过程中，是通过SpannableStringBuilder.setSpan(Object what, int start, int end, int flags)这个方法不断进行Html->Spanned转换的。例如，遇到一个a标签，则会通过下边的方法设置一个Span，在SpannabbleStringBuilder内部，Span用一个数组表示，是可以累加的。

```
// 遇到a标签头
private static void startA(SpannableStringBuilder text, Attributes attributes) {
    String href = attributes.getValue("", "href");

    int len = text.length();
    text.setSpan(new Href(href), len, len, Spannable.SPAN_MARK_MARK);
}
// a标签结束
private static void endA(SpannableStringBuilder text) {
    int len = text.length();
    Object obj = getLast(text, Href.class);
    int where = text.getSpanStart(obj);

    text.removeSpan(obj);

    if (where != len) {
        Href h = (Href) obj;

        if (h.mHref != null) {
            text.setSpan(new URLSpan(h.mHref), where, len,
                         Spannable.SPAN_EXCLUSIVE_EXCLUSIVE);
        }
    }
}
```

可以看到a标签的转换方法，实际上，a标签最后被转换成了一个`URLSpan`。

看到这里，思路就来了！实际上，`Html.fromHtml()`方法最后转换成的对象是一个`SpannableStringBuilder`，我们可以拿到这个对象的引用，然后获取所有的`URLSpan`，最后把这些`URLSpan`全部转换成可以监听的事件不就实现了吗？实际上并没有这么简单，URLSpan是一个类，继承自ClickableSpan，覆盖了其中的`onClick(View)`方法：

```
public class URLSpan extends ClickableSpan implements ParcelableSpan {

    private final String mURL;

    public URLSpan(String url) {
        mURL = url;
    }

    public URLSpan(Parcel src) {
        mURL = src.readString();
    }
    
    public int getSpanTypeId() {
        return TextUtils.URL_SPAN;
    }
    
    public int describeContents() {
        return 0;
    }

    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(mURL);
    }

    public String getURL() {
        return mURL;
    }

    @Override
    public void onClick(View widget) {
        Uri uri = Uri.parse(getURL());
        Context context = widget.getContext();
        Intent intent = new Intent(Intent.ACTION_VIEW, uri);
        intent.putExtra(Browser.EXTRA_APPLICATION_ID, context.getPackageName());
        context.startActivity(intent);
    }
}
```

这里已经默认使用了`隐式Intent`的方式打开`Uri`。**我们不能直接改变URLSpan类的实现方式，但可以继承这个类并覆盖掉它的`onClick(View)`方法，或者直接继承`ClickableSpan`。**但是，这样还是会有问题，原先的`URLSpan`早就在解析的时候存在于`SpannableStringBuilder`中的，我们需要先移除对应的`URLSpan`，然后再设置自己实现的新的`ClickableSpan`就可以了。

具体代码如下：

```
public static SpannableStringBuilder setTextLinkOpenByWebView(final Context context, String answerString) {
    if (!TextUtils.isEmpty(answerString)) {
        Spanned htmlString = Html.fromHtml(answerString);
        if (htmlString instanceof SpannableStringBuilder) {
            SpannableStringBuilder spannableStringBuilder = (SpannableStringBuilder) htmlString;
            // 取得与a标签相关的Span
            Object[] objs = spannableStringBuilder.getSpans(0, spannableStringBuilder.length(), URLSpan.class);
            if (null != objs && objs.length != 0) {
                for (Object obj : objs) {
                    int start = spannableStringBuilder.getSpanStart(obj);
                    int end = spannableStringBuilder.getSpanEnd(obj);
                    if (obj instanceof URLSpan) {
                    	//先移除这个Span，再新添加一个自己实现的Span。
                        URLSpan span = (URLSpan) obj;
                        final String url = span.getURL();
                        spannableStringBuilder.removeSpan(obj);
                        spannableStringBuilder.setSpan(new ClickableSpan() {
                            @Override
                            public void onClick(View widget) {
                                ActivityUtils.startWebviewActivity(context, url, true);
                            }
                        }, start, end, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
                    }
                }
            }
            return spannableStringBuilder;
        }
    }
    return new SpannableStringBuilder(answerString);
}

```

## 总结

TextView真的是Android里最强大的组件之一，复杂度很高，源码甚至比Activity还要多。正确的使用TextView具有意想不到的效果~文中为了解决TextView组件中的超链接使用App自带的WebView打开这个问题进行了一次探讨，最终通过hook拿到`URLSpan`，移除它并实现自己的`ClickableSpan`，最终解决问题。