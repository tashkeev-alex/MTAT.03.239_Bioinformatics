# EM-algorithm






![](EM_illustration.png)<!-- -->


# The EM-algorihm
First, we need an implementation of the EM-algorithm to estimate transcript abundances from exon-level counts and exon lengths:


```r
performEM <- function(M, k, len, NITER){
  #Original implementation provided by Ernest Turro 
	mu.trace <- list()
	X.trace <- list()

	#Initialize X and mu
	#Divide k reads equally among transcripts
	x <- k/apply(M,1,sum)
	X = c()
	for (i in c(1:ncol(M))){ X = cbind(X,x) }
	X.trace[[1]] <- X * M
	mu.trace[[1]] <- apply(X.trace[[1]],2,sum)/len #Initialize mu with its maximum likelihood estimate

	#Implementation of the EM algorithm
	for (iter in c(2:NITER)){
		mu = mu.trace[[iter-1]] #Take the latest mu values
		X = matrix(NA, ncol = ncol(M), nrow = nrow(M)) #Initalize X with empty matrix

		#The E-step (updated X_it)
		for (i in c(1:nrow(M))){
			for (t in c(1:ncol(M))){
				X[i,t] <- k[i]*M[i,t]/sum(M[i,]*mu)*mu[t]
			}
		}
		X.trace[[iter]] = X
		#The M-step (find maximum likelihood estimate of mu)
		mu.trace[[iter]] = apply(X,2,sum)/len
	}
	return(do.call(rbind, mu.trace))
}
```

## Two correctly annotated transcripts

First, let's specify the data

```r
len <- c(200, 300) # the lengths
M <- matrix(c(1,1,0,1), byrow = T, ncol = 2) #the transcripts
k <- c(200,30) # the counts for each set

#Run the EM-algorithm
mu.em = performEM(M,k,len, NITER = 1000)
tail(mu.em,1)
```

```
##           x   x
## [1000,] 0.7 0.3
```

## Two correctly annotated transcripts + 1 unexpressed transcript


```r
len <- c(200, 300, 400) # the lengths
M <- matrix(c(1,1,1,0,1,1,0,0,1), byrow = T, ncol = 3) #the transcripts
k <- c(200,30,0) # the counts for each set

#Run the EM-algorithm
mu.em = performEM(M,k,len, NITER = 1000)
tail(mu.em,1)
```

```
##           x   x             x
## [1000,] 0.7 0.3 3.454496e-126
```

## One of the true transcripts is missing, reads are captured by the transcript that is actually not expressed

```r
len <- c(200, 400) # the lengths
M <- matrix(c(1,1,0,1), byrow = T, ncol = 2) #the transcripts
k <- c(200,30) # the counts for each set

#Run the EM-algorithm
mu.em = performEM(M,k,len, NITER = 1000)
tail(mu.em,1)
```

```
##            x    x
## [1000,] 0.85 0.15
```
