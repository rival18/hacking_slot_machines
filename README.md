# SlotBot

Top of HN! https://news.ycombinator.com/item?id=18035283

So it turns out that there's a game on a specific brand of old (out of service) UK trivia machine that's basically like an extreme version of Trivial Pursuit.


Here's the pipeline:

- Capture image of slot machine screen with buttonhole camera to raspberry pi
- Process image to undo perspective shift and segment into question and answer boxes with OpenCV
- Pass processed question boxes to Google Tesseract for text recognition
- Run OCR text through a hand-designed brute-force search to get the most likely answer
- Pass answer through text-to-speech engine and into hidden earpiece

**Disclaimer: This lets you win trivia games, *provided you already have the answers*. Game legitimately aquired and not included for obvious reasons. This doesn't physically work unless you have authorized access to the machine. These machines aren't actually in circulation, you have to find them online and buy them. Be responsible, and have fun!**


### The game

The game asks you a series of general knowledge questions. It presents you with a choice of four answers, where one is correct. The more you get right, the more money you build up, until you win the jackpot.

### Designing a search algorithm

Now we have the data, the fun begins. We need to read the screen, match what we can see to a question in the data bank via a search, and read out the corresponding correct answer.

I initially tried to do this with the question data alone, ignoring the answers. Unfortunately, this doesn't work. Optical character recognition is imperfect, especially when running in real-time off a bad camera. About 30% of the characters will typically be misread. This means that the OCR-read question text is typically too garbled to identify which exact question it corresponds to; we can only narrow the search down to about 30 possible candidates.

So, we need to use the information provided by the answers to help identify which question we are looking at. This makes the search a little more tricky, but still possible.

The two basic ingredients of this search will be (i) a way to compare two strings for similarity; and, using this, (ii) a metric to rank similarity between imperfectly observed question/answer pairs and true samples from the database.

We use the [Levenschtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance) to define the similarity between two strings, defined as the minimum number of edits needed to change one string into another. Since a longer string tends to accumulate more reading errors, we'll normalize the Levenschtein distance over its length.

We form a confusion matrix of OCR answers against database answers. Taking the Frobenius inner product between this and every 4-D [permutation matrix](https://en.wikipedia.org/wiki/Permutation_matrix) will give us the metric we need. We can brute-force search over this to find the correct answer. The intuition behind this algorithm is that we're taking the dot product between the observed confusion matrix (which is noisy due to poor observability), and idealized confusion matrices (assuming perfect observability). The idealized confusion matrices take the form of permutation matrices because the four answers can appear in any permutation.

The code to carry out the search can be found in `/src/`.

### Hardware

I bought a [raspberry pi 2](https://www.raspberrypi.org/products/raspberry-pi-2-model-b/) to run the software, and used a TTS engine (Google Tesseract) to read out the answer.
