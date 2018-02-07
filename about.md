---
layout: page
title: About
permalink: /about/
---

<hr>
<div class="post-section"></div>

<style>
body {
 text-align: justify;
 text-justify: inter-word;
}
* {
    box-sizing: border-box;
}

.timeline {
    position: relative;
    max-width: 1200px;
    margin: 0 auto;
}

.timeline::after {
    content: '';
    position: absolute;
    width: 6px;
    background-color: #1E90FF;
    top: 0;
    bottom: 0;
    left: 50%;
    margin-left: -3px;
}

.container {
    position: relative;
    background-color: inherit;
    width: 50%;
}

.container::after {
    content: '';
    position: absolute;
    width: 25px;
    height: 25px;
    right: -17px;
    background-color: white;
    border: 4px solid #1E90FF;
    top: 0px;
    border-radius: 50%;
    z-index: 1;
}

.left {
    left: 0;
	padding-left: 0px 0px;
    padding-right: 0px 20px;
}

.right {
    left: 50%;
	padding-left: 0px 20px;
    padding-right: 0px 0px;
}

.right::after {
    left: -16px;
}

.content {
    padding: 20px 30px;
    background-color: white;
    position: relative;
    border-radius: 6px;
}

@media all and (max-width: 600px) {
  .timeline::after {
    left: 31px;
  }
  .container {
    width: 100%;
    padding-left: 70px;
    padding-right: 25px;
  }

  .left::after, .right::after {
    left: 15px;
  }
  .right {
    left: 0%;
  }
}
</style>

* PhD (LOL? tbd I guess)
* MSc Computer Science, University of British Columbia 2016
* B.A.Sc Engineering Science, University of Toronto 2013

I am broadly interested in systems, machine learning, economics, signal processing and finance.
I've dabbled a bit in each. Nowadays, I mostly try to keep up-to-date by reading papers on systems 
(OSDI, SOSP, SIGCOMM, EUROSYS, MOBISYS, IEEE Wireless Transactions), machine learning (NIPS, ICML), 
and economics (/r/economics, blogs). 

When I am not dabbling, you may find me sketching, sailing, swing dancing, snowboarding, 
bouldering, learning French, and travelling. I am also rumoured to write once in a while and 
strongly maintain that my current career is just a gig until a sitting Prime Minister realizes 
my talents as a speech writer. 

The timeline of my sometimes exciting, but mostly mundane life is outlined below in reverse 
chronological order.
<div class="post-section"></div>
<div class="timeline">
  <div class="container right description">	
      <div class="content">
	     <p><span style="color:blue">Graduate school.</span> Focused on distributed systems and networks. Achievements: couple of 
		 failed conference submissions on active data and stateless middleboxes. Efforts to rectify 
		 this is still ongoing.</p>
	  </div>
  </div>
  <div class="container left description">
	  <div class="content">
		 <p><span style="color:blue">Toronto.</span> My time here affirmed my longheld belief that I was destined 
		 for greatness. Unfortunately, all my peers had similar aspirations, so some of
		 the well trodden paths became a bit crowded. In an exceptional turn of events,
		 I decided to return to Vancouver to pace myself.</p> 
	  </div>
  </div>
  <div class="container right description">	
      <div class="content">
	     <p><span style="color:blue">Banker.</span> Temporary period of insanity where I decided to gain first hand insignt into 
		 the European financial crisis.</p>
	  </div>
  </div>
  <div class="container left description">
	  <div class="content">
		 <p><span style="color:blue">Undergrad.</span> Many dreams died in these 5 brief and tumultuous years. The highlight of 
		 my academic achievements was my undergraduate thesis on "Variational Inference algorithms 
		 for phase-noise, carrier frequency offset and channel estimation in MIMO-OFDM systems". 
		 However I realized too late that the quality of research is inversely proportional to the 
		 length of the title.</p>
	  </div>
  </div>
  <div class="container right description">
      <div class="content">
	  <p><span style="color:blue">Vancouver.</span> Peak suburbia.</p> 
	  </div>
  </div>
  <div class="container left description">
      <div class="content">
      <p><span style="color:blue">Shanghai</span>. Born during a pandemic during the dawn of the 
	  Chinese renaissance. The first month of my life was spent in maternity ward 
	  quarantine. It was my first personal statement of triumph over adversity.</p>
	  </div> 
  </div>
</div>
