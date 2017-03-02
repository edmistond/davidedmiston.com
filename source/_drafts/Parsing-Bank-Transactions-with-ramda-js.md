---
title: Parsing Bank Transactions with ramda.js
tags:
  - node
  - ramda
  - functional
---
Recently, I'd wanted to sort through a bunch of transaction data from my bank to figure out what our spending trends were in a couple of areas. I suppose I could've done this quite effectively with Excel or Apple Numbers, but then I said, hey, that's boring. :) I've been doing a lot of documentation and research stuff at work lately and really wanted to get my hands on a little toy project for a change of pace.

That said, I didn't want to spend *too* much time getting bogged down in stuff, so I decided to do it with node.js; node 7.6.0 was released recently and has native async/await, so it seemed like a great choice for a couple hours of hacking.

First thing to do was to get a bunch of transaction data from my bank's web site; they make it easy to download in CSV. That returns a file with the following format:

{% codeblock %}
"Date","Reference Number","Payee Name","Memo","Amount","Category Name"
{% endcodeblock %}

The payee name field seems to have a hard limit of 15 characters for most entries, unless they originated within the bank's system. Annoying and not totally descriptive, but whatever, we can deal with it.

First order of business: get the file in. I'm using [fast-csv](https://www.npmjs.com/package/fast-csv) to parse the files. The actual process of that is pretty straightforward:

{% codeblock lang:js %}
let transactions = [];

let stream = fs.createReadStream('file.csv');

csv.fromStream(stream, { headers: true })
  .on('data', (data) => { transactions.push(data); })
  .on('end', () => { processTransactions(transactions); });

let processTransactions = (transactionList) => {
  // do stuff here
};

{% endcodeblock %}

This works, but it's kind of a gross start and puts us on the road to callback hell. Since that way lies madness, let's make this use async/await instead. First, since we're dealing with some streams here, we need that function to return a promise:

{% codeblock lang:js %}
function getTransactionsFromFile(fname) {
  return new Promise(async (resolve, reject) => {
    // do stuff here
  });
}
{% endcodeblock %}

Now, let's fill this in a bit. If the file doesn't exist, then we should reject the promise, essentially throwing an exception: 

{% codeblock lang:js %}
function getTransactionsFromFile(fname) {
  return new Promise(async (resolve, reject) => {

    if (!fs.existsSync(fname)) {
      return reject('file does not exist!');
    }

    // do stuff here
  });
}
{% endcodeblock %}

So far, so good. Now, since this is a small file, rather than process the stream events individually, I'd rather just append them all to a collections object, read 'em in, and then return them all at once. If you were doing this on a huge file, that's probably a bad idea. :) However, for ~500 transactions or so, it's no big deal. So let's do that: we're going to read in the file, append all the transactions into an array, then finally resolve the promise and return the data once we hit the end of the file. Along the way, we're also going to reject the promise if the CSV parsing decides to barf:

{% codeblock lang:js %}
function getTransactionsFromFile(fname) {
  return new Promise(async (resolve, reject) => {

    if (!fs.existsSync(fname)) {
      return reject('file does not exist!');
    }

    let transactions = [];
    let stream = fs.createReadStream(fname);

    csv.fromStream(stream, { headers: true })
      .on('data', (data) => { transactions.push(data); })
      .on('error', (err) => { return reject(error); })
      .on('end', () => { return resolve(transactions); });
  });
}
{% endcodeblock %}

So... let's test this puppy out!

{% codeblock lang:js %}
let transactionList = await getTransactionsFromFile('./Huntington.csv');
processTransactions(transactionList);

// > node parse.js
// /Users/edmistond/source/finance/parse.js:6
// let transactionList = await getTransactionsFromFile('./data.csv');
//                             ^^^^^^^^^^^^^^^^^^^^^^^
// SyntaxError: Unexpected identifier
{% endcodeblock %}

What just happened here?

Turns out that you can't just await a function *anywhere* - you have to call them inside an async function. So what does that look like? Oh, and let's define a basic processTransactions function so we can see some data.

{% codeblock lang:js %}
async function run() {
  try {
    let transactionList = await getTransactionsFromFile('./data.csv');
    processTransactions(transactionList);
  }
  catch (ex) {
    console.log('I caught an error:', ex);
    return;
  }
}

let processTransactions = (transactionList) => {
  console.log('got', transactionList.length, 'transactions');
};


run();

// > node parse.js
// got 478 transactions
{% endcodeblock %}

Excellent. Data. Now how to parse it?

Traditionally, this sort of thing would involve writing a bunch of looping code to iterate through the transactions array, examining each one, and possibly passing it into another array if it was a transaction that was of interest to us. Then we'd write more loops to do other operations, like summing up all the transactions of a given type. Pretty straightforward, but honestly, also exceptionally tedious. 

This is where [Ramda](http://ramdajs.com/) comes in. You can use [lodash](http://lodash.com) as well, but I've become rather fond of Ramda over the past year or so. Though I wouldn't go adding it willy-nilly into existing projects that already used lodash, it's been my library of choice for doing map/filter/reduce/sort type operations in my personal projects. I like that it automatically curries functions when you don't supply all the arguments, and that it in fact encourages this use by making the collection/array object the last argument supplied to the function.

I also particularly like this approach because I think it's more explicit. If you're using a loop, anyone else reading the code has to take the time to inspect the loop and understand what the code in it is doing. When I see a `.map` function called, I immediately know "Oh, this is transforming data." Yes, it can be a little more obtuse up front if someone else isn't familiar with the concept, but I think it's pretty straightforward to understand and much clearer about what's going on once you know that.


So let's say we want to figure out how much we spent on the dog, and also how much we spent eating out. First, we need to tag our transactions with some additional information:

* Was it income or an expense?
* Was it a restaurant?
* Was it pet-related?

We don't *have* to do it this way, but I'm going to run the transactions array through a map operation that transforms them by adding additional data to them. First we need a function that takes a single transaction, inspects the payee and payment amounts, and tags them with some data:

{% codeblock lang:js %}
const restaurantPayees = ["MCDONALDS", "CHIPOTLE", "JACKS DELI"];
const petPayees = ["CAMP BOW WOW", "VET", "PET STORE", "GROOMER"];

const tagTransaction = (transaction) => {
  if(R.contains(transaction['Payee Name'], restaurantNames)) {
    transaction.isRestaurant = true;
  }
  else if (R.contains(transaction['Payee Name'], petPayees)) {
    transaction.isPet = true;
  }

  transaction.isIncome = parseFloat(transaction.Amount) > 0.0;
  transaction.Amount = Math.abs(parseFloat(transaction.Amount));

  return transaction;
};
{% endcodeblock %}

And then, just to check it's working right, let's add a quick filter operation on the pet transactions to see how many there were.

{% codeblock lang:js %}
const processTransactions = (transactionList) => {
  const taggedTransactions = R.map(tagTransaction, transactionList);

  const petTransactions = R.filter(t => t.isPet, taggedTransactions);
  console.log('You had', petTransactions.length, 'purchases for the dog');
};

// > node parse.js
// You had 10 purchases for the dog
{% endcodeblock %}

But wait! We actually have an opportunity here to use currying. In short - currying is calling a function without supplying all of its values, and getting back in turn another function that you can call repeatedly against different values, using the same input value. Let's see what it looks like!

{% codeblock lang:js %}
const tagTransaction = (transaction) => {
  const checkPayeeMatch = R.contains(transaction['Payee Name']);

  transaction.isRestaurant = checkPayeeMatch(restaurantPayees);
  transaction.isPet = checkPayeeMatch(petPayees);

  transaction.isIncome = parseFloat(transaction.Amount) > 0.0;
  transaction.Amount = Math.abs(parseFloat(transaction.Amount));

  return transaction;
};

// > node parse.js
// You had 10 purchases for the dog
{% endcodeblock %}

As you can see, we built a curried `checkPayeeMatch` function by calling `R.contains` and only supplying the payee name field; we can then apply it to both the restaurants and pets payee arrays to see if there were any matches simply by calling that function.
