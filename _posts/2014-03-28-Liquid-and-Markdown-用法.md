---
title: "Liquid"
date: 2014-01-01 00:00:00 +0800
categories: [Blog, Example]
tags: [example]
comments: true
toc: true
---

###Liquid
- 代码插入高亮显示

{% highlight ruby %}
def show
    @widget = Widget(params[:id])
    respond_to do |format|
        format.html # show.html.erb
        format.json { render json: @widget }
    end
end
{% endhighlight %}
{% highlight c %}
static int sw_upgradeEvent_observer_start()
{
	m_upgradeevent_thrd = sw_thrd_open("tUpgradeEvent",100,0,64*1024,upgrade_event_proc,0,0);
	if( m_upgradeevent_thrd == NULL)
		return -1;

	sw_thrd_resume(m_upgradeevent_thrd);
	return 0;
}
{% endhighlight %}

