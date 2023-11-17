# C++ Move Semantics

Created: October 29, 2023 6:50 PM
Tags: CS, TODO

My Junior year of school I sat down with a Microsoft interviewer. We chatted for a while, I thought things were going well, then he throws this curveball:

************************What are L and R-values?************************

I didn’t know, and the interview moved on (I didn’t get the job, but I seriously doubt it was because I didn’t understand this C++ nuance - how many college undergraduates know this?). Four years later I find this Reddit [post](https://www.reddit.com/r/cpp/comments/ek32mq/c_move_semantics_the_complete_guide/) on C++ move semantics. Shock and awe - L and R values!

I haven’t used CPP in my career since graduating and I find myself growing rusty, so I decided to learn a bit about these move semantics, “*a* *********************************fundamental feature of modern CPP”*********************************. I’m also somewhat jaded after that interview, so I’d like to be prepared should we have to do battle once more.

## So what ARE L and R values?