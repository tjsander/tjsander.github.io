---
layout: post
title: My First Jekyll Post!
---

#### Testing 12345...

Hello world! Here is some C code!

{% highlight C %}
void *sieve_primes(void *arg)
{
	unsigned int k;
	unsigned int m, i;

	struct data *d = (struct data*)arg;
	int * list = d->list;
	unsigned int n = d->n;
	k = 1;

	while (k < sqrt(n)) {
		m = k+1;
		i = 2;

		pthread_mutex_lock(&mutex_q);
		if (m < topstack) m = topstack+1;
		while ((m < n) && ISBITSET(list, m)) m++;
		if (m > topstack) topstack = m;
		SETBIT(list, m);
		pthread_mutex_unlock(&mutex_q);

		while ((long)m*i < n) {
			if (!ISBITSET(list, m*i)) SETBIT(list, m*i);
			i++;
		}

		pthread_mutex_lock(&mutex_r);
		CLEARBIT(list, m);
		pthread_mutex_unlock(&mutex_r);
		k = m;
	}
	pthread_exit((void*)0);
}
{% endhighlight %}

And here is a picture:
![IGNORE ME](/assets/images/ignore.jpg "IGNORE ME")

Markdown is pretty alright.