# Why I Built It This Way

So, you might be wondering why I picked certain tools and approaches for this email sorter thingy. Here's the lowdown, straight from my (digital) whiteboard:

**To understand the functionality of this Git repository in detail, please refer to the [SOLUTION DETAILS](Solution.md) file.**

**To understand the reasoning behind the design choices and the "why" of this solution, continue reading below**

**Gmail:**

Think of it this way: I was trying to teach my computer to read my own emails first, just to get the hang of things. Since my personal email is on Gmail, it was the easiest way for me to plug in and start pulling emails. I didn't have access to a company Outlook account or the fancy Microsoft Graph API, which would've been the go-to if I were dealing with a real work mailbox. So, Gmail was just the quick and dirty way to get started and show the idea working.

**Why LLMs for Figuring Out What Emails Are About?**

Imagine trying to teach a computer to sort emails into "urgent," "just a question," or "marketing fluff." Usually, you'd need a big pile of emails that are *already* labeled correctly so the computer can learn. But guess what? I didn't have that nicely labeled pile!

That's where these super-smart AI models (LLMs) come in. They've read tons of text on the internet, so they're pretty good at understanding what things *mean* even if they haven't seen that exact email before. It's like asking a really well-read friend to guess what a book is about just by the cover and a few sentences.

**Cool Idea for the Real Deal:** What would be super cool in a real company is if I could grab all the old emails that people *already* sorted manually. The email content would be like the book, and whoever they sent it to would be a clue about what the email was about ("sent to the billing team? probably about billing!"). I could also have people quickly tag a few emails themselves to make the AI even smarter and more specific to the company's needs. That way, it wouldn't just be a general guess from a big AI brain, but something that really understands *your* emails.

**What's with My Weird Email Categories?**

Okay, so the categories I used ("technology," "News," "Job search," etc.)? Those are just the kinds of emails I get in my personal inbox! It was just a way to show that the system *can* sort emails into different buckets.

**For a Real Company:** The categories would be totally different and based on what your clients actually email about. Think things like "Problem with my order," "Question about my account," "I want to complain," or "Just saying hi" (though maybe we can ignore those!).

**Bonus point** - the configuration json files are not incuded in this repo - because they are my personal files :)

**BigQuery? Sounds Fancy!**

So, for storing all the emails and the computer's guesses about them, I needed a place to put it all. Since I was already playing around with Google stuff (because of the Gmail connection), BigQuery was just a convenient place to dump the data. It's like a giant digital spreadsheet in the cloud that can handle a lot of info.

**Bottom Line:** You could totally use other places to store this stuff – Amazon has its own version, there are regular databases, all sorts of things. The main idea was just to have a place to keep all the email history so I can look at trends later on.

Hopefully, that makes a bit more sense about why I made the choices I did! It was a mix of what was easy to use for a quick demo and a peek at how I could build this out for a real business.
