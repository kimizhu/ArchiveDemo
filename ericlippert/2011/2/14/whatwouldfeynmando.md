# What would Feynman do?

[Eric Lippert](https://social.msdn.microsoft.com/profile/Eric%20Lippert) 2/14/2011 6:29:00 AM

-----

No one I know at Microsoft asks those godawful "lateral-thinking puzzle" interview questions anymore. Maybe someone still does, I don't know. But rumour has it that a lot of companies are still following the Microsoft lead from the 1990s in their interviews. In that tradition, I present a sequel to [Keith Michaels' 2003 exercise in counterfactual reasoning](http://www.sellsbrothers.com/Posts/Details/12395). Once more, we dare to ask the question "*how well would the late Nobel-Prize-winning physicist Dr. Richard P. Feynman do in a technical interview at a software company?*"

**Interviewer**: Now we come to the part of the interview where we test your creative thinking. Don't think too hard about it; just apply common sense and explain your reasoning. Here's the problem.

> *You are in a room with three switches that each control a different light fixture in another room. You cannot see from the switch room into the lamp room. Your task is to determine which switches control which light fixtures, but you may only go into the room with the lights once. How do you determine which switch controls which light?*

**RPF**: That seems straightforward. I could obtain a number of large mirrors, and, if necessary, a telescope. I enter the room with the lights once and position the mirror so that it reflects all three lights out the door of the room.  I continue placing mirrors, aligning them as necessary to reflect the photons emitted by the lights until I am back in the room with the switches. Now I can see the lights, possibly through the telescope if the distance is large, and I can toggle the switches on and off so as to determine which light is controlled by which switch.

**Interviewer**: Um. Yeah, I suppose that would work. But what if you didn't have big mirrors, or couldn't align them well enough?

**RPF**: Then I could obtain an inexpensive digital video camera and put it on a dolly with a sufficiently long rope attached to it.  I could put the video camera in the room with the lights, turn it on, and then take the other end of the rope back to the room with the switches. I'd then play with the switches for a while and take notes on which switches I flipped at what time.  Then I'd haul the camera on its dolly back into the switch room and review the recording. By correlating my notes of what switches were flipped at what time with the recording of the lights, I could correlate lights to switches.

**Interviewer**: I forgot to mention that once you enter the room with the lights, you are not allowed to come back to the room with the switches.

**RPF**: That is an unusual constraint that perhaps you ought to have mentioned earlier, but I'll go with it. In that case I would take a different approach. But first I'll need more information. Can I assume that the lights and the switches are correctly wired according to the National Electric Code of the United States? That is, that the switches interrupt the hots, not the neutrals, that the switches are standard-duty switches rated to interrupt 15 amps of 120 volt alternating current, and so on?

**Interviewer**: Yeah, I guess so.

**RPF**: And these are *single* switches? Or is it possible that the switches are part of a *multi-location* switch, like you see in houses where there are two light switches for the same light, say, at the top and bottom of the stairs?

**Interviewer**: Does it matter?

**RPF**: Certainly it matters\! You're asking me a question about correctly deducing the properties of a 120 volt electrical system. The resistance across a well-grounded human being is, I don't know, call it 1000 ohms, and we know that current is equal to voltage divided by resistance. That means that an accidental shock could put a current of 120 milliamps across that human, which is within the range that will stop someone's heart. I presume you know the details of the system you are asking me to diagnose; the safety precautions I'm going to describe will be different depending on the known and unknown aspects of the electrical system.

**Interviewer**: Right. Suppose they are just normal switches, nothing fancy.

**RPF**: Great. Are the three switches all in one triple-wide junction box, as those switches over there on your wall that control your lights in your office are, or are there three different junction boxes, one for each switch?

**Interviewer**: The former.

**RPF**: As I'm sure you know, there are two standard ways of wiring three single-location switches as you describe. The first is to bring the hot and neutral return wires from the power source to the triple-switch box, then split the hot into three to power each switch, and then run three switched hot wires and three unswitched neutral wires, one pair to each light. The second way is to do the opposite: run the hot and neutral from the panel to each light, and then run a pair of hot wires from each light to its switch. The switch joins together the two hot wires so that one of the hot wires is unswitched and the other is switched. The lights are then energized by the switched hot. Which of these two standard configurations did the electrician who wired this system use?

**Interviewer**: I don't think it matters. But I don't actually know how to wire a light switch.

**RPF**: It seems odd that you would ask me a question about deducing properties an electrical system but not know the details of that electrical system.

As a simplifying assumption let's suppose that the system is wired with the first configuration I described. That is, there are "line" hot and neutral wires in the switch box, and that the hot is interrupted by the three switches. This means that when I remove the cover of the light switch, I can easily determine which hot wire is coming from the panel, and which switched hot wires are going to the fixtures. Before I remove the cover of course I would find the electrical panel and de-energize the circuit that powers the switch.  If necessary, I could simply de-energize every circuit, if for some reason I could not reliably determine which breaker corresponded to the circuit I was about to work on. I would also inform everyone in the vicinity that there was a breaker switched off and that I was working on the mains. I'd probably post a sign that said to not turn the power back on, and if it was equipped with a lock, I'd lock the breaker in the off position and pocket the key. I've been shocked enough times already in this life; I'd rather not take a chance on being electrocuted for the purposes of your exercise.

At this point I note that the problem you pose is, in a trivial sense, solved.

**Interviewer**: What on earth are you talking about?

**RPF**: The problem was to determine which switch controls which lights. With the mains power turned off, in a sense *none* of the switches control *any* of the lights. Any of the switches can be in any position and none of the lights will go on. But I think that's not the sort of solution you had in mind.

**Interviewer**: You are correct; that's not where I was going with this.

**RPF**: Now that the power is off I can safely disassemble the light switch junction box and disconnect the three switched hot wires from their switches. I would obtain a piece of standard NM-14/3 copper wire long enough to go from the switch room to the lamp room. Attach the white conductor to the disconnected switched hot wire of the first switch, the black conductor to the second and the red conductor to the third. I'd then carry the other end of my wire to the room with the lights, which should all be off. I'd remove the lamps from the fixtures, and then use the conductor as a *continuity tester*. By using a nine-volt battery and a DC volt meter, I can determine when each of the three conductors completes a circuit with the hot portion of the light fixture. I'd then know which lamp corresponds to which switch.

**Interviewer**: What if it was infeasible to obtain a piece of conductor that long?

**RPF**: By the statement of the problem there already are at least *three* conductors that long going between the switches and the lights, so it was feasible for someone already. Unless you are implying that the light switches are actually part of some sort of radio control system. Again, this seems like a fact about the system that you ought to have mentioned earlier; you did say that these were "nothing fancy" 120 volt AC switches.

**Interviewer**: They're just normal switches. But I think you've forgotten something else.

**RPF**: Yes, I see your objection. I asked earlier if the *switches* were rated to 15 amps at 120 volts but I did not ask if the *light fixtures* were. If the light fixtures are low-voltage fixtures then there is an *AC transformer* sitting between the high-voltage switched hot that I've got my continuity tester on and the low-voltage fixtures. My nine volt direct current continuity tester isn't going to give me a good result in that case.

**Interviewer**: Actually I was going to say that since you're not allowed to go back into the room with the switches, you'll be leaving this scenario with the switches disassembled and the breaker locked in the off position.

**RPF**: You make an excellent point. I should come up with a solution that leaves the switches assembled, since I'm not allowed to come back.

**Interviewer**: Yes, you should. Can you?

**RPF**: Suppose instead of attaching a continuity tester after I disassemble the light switches, I simply swap out all of the light switches for dimmers. I set the first dimmer to low, the second dimmer to medium and the third dimmer to high. I restore the power, and now when I go to the other room, I know which light corresponds to which switch by observing their relative brightness.

**Interviewer, relieved**: Now you're getting somewhere. But...

**RPF**: Yes, again I see the problem you're about to point out. If the lamps are *fluorescent* then two of them will be off or flickering, only the one on "high" will be on, so I have possibly only determined which light is the "high" dimmer switch. And if the lamps are incandescent, then their *brightness* could be different depending on their *wattage*. It was not a condition of the problem scenario that the three lamps all be *incandescent bulbs of the same voltage and wattage*. I haven't actually solved the problem. Instead I could remove the lamps and test each fixture's hot-to-neutral potential with an AC volt meter to see which one has the high, medium or low voltage. Though that again is assuming that there aren't transformers in there that are changing the voltages.

**Interviewer**: Forget about measuring the voltage already\! Suppose you can't reach the fixture to measure its voltage.

**RPF**: Again, I must point out that it seems very odd to ask a question about diagnosis of an electrical system while not allowing the diagnostician to use common electrical tools. But anyway, you said that I was on the right track, so let's go with that. We know that modern dimmers do not put a *variable resistance* across the AC signal; rather, they selectively "cut out" a variable-sized portion of the wave and leave the rest of the cycle in its normal size and shape. We could build a device that works analogously to a dimmer, but much slower. The device could have a couple of rotating cams that flip a switch on and off once a second. Now we need not disassemble any of the switches, or cut the power at the panel. We attach the device to the first switch, flip the second switch off, and the third switch on. Since we have already established that the switches are single-location switches that have been wired correctly according to the NEC, we know that the switch in the "up" position is energizing its lamp and the one in the down position is off.  Now we go into the other room. The lamp that is off is controlled by the third switch, the lamp that is on is controlled by the second, and the one that is flipping on and off every second is controlled by the first. This system will work no matter what kind of lamps are in the fixtures, provided of course that they are good lamps, not burned out.

**Interviewer**: Well I suppose that would work. All of your solutions so far require some kind of equipment. Could you solve the problem without building anything or using special tools, by taking advantage of some other factor?

**RPF**: Like what?

**Interviewer**: Like, that lamps produce effects other than lighting a room.

**RPF**: For example?

**Interviewer, exasperated**: You could turn two switches on and one off. Then wait a minute, and turn the third switch off. When you go to the other room, the lamp controlled by the first switch will be on, the lamp controlled by the second switch will be off and cold, and the lamp controlled by the third switch will be off and hot. That seems a lot easier than all this rigamarole about disassembling the switches or building custom equipment.

**RPF**: How am I to measure the heat of the lamps without special equipment? You just said that I couldn't reach them.

**Interviewer**: Um. Yes, I suppose I did say that.

**RPF**: I can see a number of additional problems with your heuristic. You haven't specified how far it is between the rooms, but have several times implied that it is a considerable distance; I can't see the light from the switch room, I can't align my mirrors and I can't bring a conductor that long for continuity testing all imply considerable distance between the switch room and the light room. The time it takes me to get from one room to another can give the third lamp time to cool. The third lamp might not be very hot to begin with; if the fixtures are fluorescent bulbs, as they are in this building, or modern LED bulbs, then their heat output is far lower than an incandescent bulb. We also haven't specified where this scenario takes place. If it is in a very hot climate, like Los Alamos in the summer, both de-energized lamps could reasonably be warm to the touch, and if it is in Alaska in the winter in an uninsulated room then both could reasonably be cool by the time I get there. Your proposed heuristic depends upon a number of conditions that were not given in the problem. And it is in general a bad idea to test whether something is extremely hot by touching it.

**Interviewer**: Well I think that concludes this portion of the interview. Before we let you go for the day do you have any questions for me about this company, this team or the job?

**RPF**: Yes. When you build software algorithms, do you build systems using well-established software engineering principles to produce software that conforms to industry standards and practices?

**Interviewer**: Of course.

**RPF**: And do you use software analysis tools, like profilers, debuggers, theorem provers, and so on, to facilitate detection and diagnosis of flaws?

**Interviewer**: Yes, again, of course we do.

**RPF**: Then why would you ask an interview question that tests my willingness to abandon industry-standard, well-established techniques that use common electrician's tools to determine continuity of a portion of an electrical system? And why is the solution you were clearly driving me towards one which takes advantage of an *undocumented and unreliable epiphenomenon*? Does your team usually write code whose correctness relies upon undocumented and unreliable correlations, correlations whose magnitudes can vary widely as a result of implementation details?

**Interviewer**: Thanks for coming in Dr. Feynman. We'll be in touch.

