##DPM Code

 - Model format documentation in doucumentation.txt

Model paraments:

    filters:
    rules:
    symols:
    start:

**'model.thresh' defines a threshold that can be used in the 'detect' function to obtain a high recall rate.**



>P. Felzenszwalb, R. Girshick, D. McAllester, D. Ramanan
Object Detection with Discriminatively Trained Part Based Models
IEEE Transactions on Pattern Analysis and Machine Intelligence, Vol. 32, No. 9, Sep. 2010

##DPM平台移植


**因为DPM需要在Linux下运行，所以将其运行在windows下需要进行修改**

 - DPM版本：voc-release4.01
 - PC：windows 7 64位
 - 编译器：Microsoft Visual C++ 2013 Professional


### How to run objection Detection with Discriminatively Trained Part Based Models

 - 修改dt.cc,fconv.cc,features.cc,getdetections.cc,resize.cc的后缀.cc为.cpp;
 - dt.cpp 中 添加

---
    #define int32_t int

  - feature.cpp 加入 round()如下，必要的话

---

  - resize.cpp 加入和修改

---

    #define bzero(a,b) memset(a,0,b)

    //如果必要,需要添加这段代码
    int round(float a)
    { 
           float tmp = a - (int)a;
           if( tmp >= 0.5 )
               return (int)a + 1;
           else
               return (int)a; 
    }

---

    //alphainfo ofs[len];
    struct alphainfo *ofs = (struct alphainfo *)malloc(sizeof(struct alphainfo)*len);



---
    free(ofs);//注意释放


   - compile.m 配置

---
    mex -O resize.cpp
    mex -O dt.cpp
    mex -O features.cpp
    mex -O getdetections.cpp
    mex -O fconv.cpp


   - 编译过后就可以运行demo.m


### How to train models of Object Detection with Discriminatively Trained Part Based Models

 - 首先下载voc的数据库和相应的VOCdevkit。（注意把数据也放在VOCdevkit的目录中）

 - pascal_data.m文件添加一段

---
    numpos = numpos+1;
    %
    if exist([VOCopts.datadir rec.imgname])==0
    continue;
    end
    %
    pos(numpos).im = [VOCopts.datadir rec.imgname];
      
 - pascal.m 文件修改

---
    %unix(['rm ' tmpdir cls '.dat']);
    system(['del ' tmpdir cls '.dat']);

  - rewritedat.m 修改

---
    % unix(['mv ' datfile ' ' oldfile]);
    movefile(datfile,oldfile);

---

	% unix(['rm ' oldfile]);
	delete(oldfile);

---
		
	% unix(['cp ' inffile ' ' oldfile]);
	system(['copy ' inffile ' ' oldfile]);

---

修改原因：训练时会产生越界


	% sort indexes so we never have to seek before the current position
	% I = sort(I);
	% 
	% pos = 1;
	% for i = 1:length(I)
	%   cnt = I(i)-pos;
	%   while cnt > 0
	%     % + 2 to include the num non-zero blocks and example length
	%     info = fread(fin, labelsize+2, 'int32');
	%     dim = info(end);
	%     fseek(fin, dim*4, 0);
	%     cnt = cnt - 1;
	%   end
	%   y = fread(fin, labelsize+2, 'int32');
	%   dim = y(end);
	%   x = fread(fin, dim, 'single');
	%   fwrite(fout, y, 'int32');
	%   fwrite(fout, x, 'single');
	%   pos = I(i)+1;
	% end
	
	I = sort(I);
	
	pos = 1;
	for i = 1:length(I)
	  cnt = I(i)-pos;
	  while cnt > 0
	    % + 2 to include the num non-zero blocks and example length
	    info = fread(fin, labelsize+2, 'int32');
	    if length(info) == 0
	        dim = 0;
	    else
	        dim = info(end);
	    end
	    %dim = info(end);
	    fseek(fin, dim*4, 0);
	    cnt = cnt - 1;
	  end
	  y = fread(fin, labelsize+2, 'int32');
	  %//!
	  if length(y) == 0
	      dim = 0;
	  else
	      dim = y(end);
	  end
	  x = fread(fin, dim, 'single');
	  fwrite(fout, y, 'int32');
	  fwrite(fout, x, 'single');
	  pos = I(i)+1;
	end

   - train.m 修改

---
	   
	% cmd = sprintf('./learn %.6f %.6f %s %s %s %s %s %s %s %s %s', ...
	%                  C, J, hdrfile, datfile, modfile, inffile, lobfile, ...
	%                  cmpfile, objfile, cachedir, logtag);
	    
    cmd = sprintf('learn %.6f %.6f %s %s %s %s %s %s %s %s %s', ...
                   C, J, hdrfile, datfile, modfile, inffile, lobfile, ...
                   cmpfile, objfile, cachedir, logtag);

   - procid.m 修改

---

	% i = strfind(d, '/');
	i = strfind(d, '\');


   - learn.cc 变成 learn.cpp 建一个工程进行编译，修改的代码如下，编译后的.exe 文件放置于 voc-release4.01文件夹下

---
	
	#include <stdio.h>
	#include <stdlib.h>
	#include <string>
	#include <math.h>
	#include <time.h>
	#include <errno.h>
	#include <fstream>
	#include <iostream>
	//#include "stdafx.h"
	
	using namespace std;
	
	/*
	* Optimize LSVM objective function via gradient descent.
	*
	* We use an adaptive cache mechanism.  After a negative example
	* scores beyond the margin multiple times it is removed from the
	* training set for a fixed number of iterations.
	*/
	
	// Data File Format
	// EXAMPLE*
	// 
	// EXAMPLE:
	//  long label          ints
	//  blocks              int
	//  dim                 int
	//  DATA{blocks}
	//
	// DATA:
	//  block label         float
	//  block data          floats
	//
	// Internal Binary Format
	//  len           int (byte length of EXAMPLE)
	//  EXAMPLE       <see above>
	//  unique flag   byte
	
	// number of iterations
	
	/*#ifndef DRAND48_H
	#define DRAND48_H
	
	#include <stdlib.h>  */
	
	//#define m 0x100000000LL  
	//#define a 0x5DEECE66DLL  
	//static unsigned long long seed = 1;
	//#endif
	//#define Infinity 1.0+308
	
	
	#define ITER 10e6
	
	// minimum # of iterations before termination
	#define MIN_ITER 5e6
	
	// convergence threshold
	#define DELTA_STOP 0.9995
	
	// number of times in a row the convergence threshold
	// must be reached before stopping
	#define STOP_COUNT 5
	
	// small cache parameters
	#define INCACHE 25
	#define MINWAIT (INCACHE+25)
	#define REGFREQ 20
	
	// error checking
	#define check(e) \
	(e ? (void)0 : (printf("%s:%u error: %s\n%s\n", __FILE__, __LINE__, #e, strerror(errno)), exit(1)))
	
	// number of non-zero blocks in example ex
	#define NUM_NONZERO(ex) (((int *)ex)[labelsize+1])
	
	// float pointer to data segment of example ex
	#define EX_DATA(ex) ((float *)(ex + sizeof(int)*(labelsize+3)))
	
	// class label (+1 or -1) for the example
	#define LABEL(ex) (((int *)ex)[1])
	
	// block label (converted to 0-based index)
	#define BLOCK_IDX(data) (((int)data[0])-1)
	
	// set to 0 to use max-component L2 regularization
	// set to 1 to use full model L2 regularization
	#define FULL_L2 0
	
	#define MNWZ 0x100000000  
	#define ANWZ 0x5DEECE66D  
	#define CNWZ 0xB16 
	#define INFINITY_my 0xFFFFFFFFF
	
	int labelsize;
	int dim;
	
	static unsigned long long seed = 1;
	
	double drand48(void)
	{
		seed = (ANWZ * seed + CNWZ) & 0xFFFFFFFFFFFFLL;
		unsigned int x = seed >> 16;
		return  ((double)x / (double)MNWZ);
	}
	
	//static unsigned long long seed = 1;
	
	void srand48(unsigned int i)
	{
		seed = (((long long int)i) << 16) | rand();
	}
	
	// comparison function for sorting examples 
	int comp(const void *a, const void *b) {
		// sort by extended label first, and whole example second...
		int c = memcmp(*((char **)a) + sizeof(int),
			*((char **)b) + sizeof(int),
			labelsize*sizeof(int));
		if (c)
			return c;
	
		// labels are the same  
		int alen = **((int **)a);
		int blen = **((int **)b);
		if (alen == blen)
			return memcmp(*((char **)a) + sizeof(int),
			*((char **)b) + sizeof(int),
			alen);
		return ((alen < blen) ? -1 : 1);
	}
	
	// a collapsed example is a sequence of examples
	struct collapsed {
		char **seq;
		int num;
	};
	
	// the two node types in an AND/OR tree
	enum node_type { OR, AND };
	
	// set of collapsed examples
	struct data {
		collapsed *x;
		int num;
		int numblocks;
		int numcomponents;
		int *blocksizes;
		int *componentsizes;
		int **componentblocks;
		float *regmult;
		float *learnmult;
	};
	
	// seed the random number generator with an arbitrary (fixed) value
	void seed_rand() {
		srand48(3);
		//srand(3);
	}
	
	static inline double min(double x, double y) { return (x <= y ? x : y); }
	static inline double max(double x, double y) { return (x <= y ? y : x); }
	
	// compute the score of an example
	static inline double ex_score(const char *ex, data X, double **w) {
		double val = 0.0;
		float *data = EX_DATA(ex);
		int blocks = NUM_NONZERO(ex);
		for (int j = 0; j < blocks; j++) {
			int b = BLOCK_IDX(data);
			data++;
			double blockval = 0;
			for (int k = 0; k < X.blocksizes[b]; k++)
				blockval += w[b][k] * data[k];
			data += X.blocksizes[b];
			val += blockval;
		}
		return val;
	}
	
	// return the value of the object function.
	// out[0] : loss on negative examples
	// out[1] : loss on positive examples
	// out[2] : regularization term's value
	double compute_loss(double out[3], double C, double J, data X, double **w) {
		double loss = 0.0;
	#if FULL_L2
		// compute ||w||^2
		for (int j = 0; j < X.numblocks; j++) {
			for (int k = 0; k < X.blocksizes[j]; k++) {
				loss += w[j][k] * w[j][k] * X.regmult[j];
			}
		}
	#else
		// compute max norm^2 component
		for (int c = 0; c < X.numcomponents; c++) {
			double val = 0;
			for (int i = 0; i < X.componentsizes[c]; i++) {
				int b = X.componentblocks[c][i];
				double blockval = 0;
				for (int k = 0; k < X.blocksizes[b]; k++)
					blockval += w[b][k] * w[b][k] * X.regmult[b];
				val += blockval;
			}
			if (val > loss)
				loss = val;
		}
	#endif
		loss *= 0.5;
	
		// record the regularization term
		out[2] = loss;
	
		// compute loss from the training data
		for (int l = 0; l <= 1; l++) {
			// which label subset to look at: -1 or 1
			int subset = (l * 2) - 1;
			double subsetloss = 0.0;
			for (int i = 0; i < X.num; i++) {
				collapsed x = X.x[i];
	
				// only consider examples in the target subset
				char *ptr = x.seq[0];
				if (LABEL(ptr) != subset)
					continue;
	
				// compute max over latent placements
				int M = -1;
				double V = -INFINITY_my;
				//double V = -NWZ;
				for (int m = 0; m < x.num; m++) {
					double val = ex_score(x.seq[m], X, w);
					if (val > V) {
						M = m;
						V = val;
					}
				}
	
				// compute loss on max
				ptr = x.seq[M];
				int label = LABEL(ptr);
				double mult = C * (label == 1 ? J : 1);
				subsetloss += mult * max(0.0, 1.0 - label*V);
			}
			loss += subsetloss;
			out[l] = subsetloss;
		}
	
		return loss;
	}
	
	// gradient descent
	void gd(double C, double J, data X, double **w, double **lb, char *logdir, char *logtag) {
		ofstream logfile;
		string filepath = string(logdir) + "/learnlog/" + string(logtag) + ".log";
	
		/*char* filepath;
		strcat(filepath,logdir);
		strcat(filepath,"/learnlog/");
		strcat(filepath,logtag);
		strcat(filepath,"/log");*/
	
		logfile.open(filepath.c_str());
		//logfile.open(filepath);
		logfile.precision(14);
		logfile.setf(ios::fixed, ios::floatfield);
	
		int num = X.num;
	
		// state for random permutations
		int *perm = (int *)malloc(sizeof(int)*X.num);
		check(perm != NULL);
	
		// state for small cache
		int *W = (int *)malloc(sizeof(int)*num);
		check(W != NULL);
		for (int j = 0; j < num; j++)
			W[j] = INCACHE;
	
		double prev_loss = 1E9;
	
		bool converged = false;
		int stop_count = 0;
		int t = 0;
		while (t < ITER && !converged) {
			// pick random permutation
			for (int i = 0; i < num; i++)
				perm[i] = i;
			for (int swapi = 0; swapi < num; swapi++) {
				int swapj = (int)(drand48()*(num - swapi)) + swapi;
				//int swapj = (int)(rand()*(num-swapi)) + swapi;
				int tmp = perm[swapi];
				perm[swapi] = perm[swapj];
				perm[swapj] = tmp;
			}
	
			// count number of examples in the small cache
			int cnum = 0;
			for (int i = 0; i < num; i++)
				if (W[i] <= INCACHE)
					cnum++;
	
			int numupdated = 0;
			for (int swapi = 0; swapi < num; swapi++) {
				// select example
				int i = perm[swapi];
	
				// skip if example is not in small cache
				if (W[i] > INCACHE) {
					W[i]--;
					continue;
				}
	
				collapsed x = X.x[i];
	
				// learning rate
				double T = min(ITER / 2.0, t + 10000.0);
				double rateX = cnum * C / T;
	
				t++;
				if (t % 100000 == 0) {
					double info[3];
					double loss = compute_loss(info, C, J, X, w);
					double delta = 1.0 - (fabs(prev_loss - loss) / loss);
					logfile << t << "\t" << loss << "\t" << delta << endl;
					if (delta >= DELTA_STOP && t >= MIN_ITER) {
						stop_count++;
						if (stop_count > STOP_COUNT)
							converged = true;
					}
					else if (stop_count > 0) {
						stop_count = 0;
					}
					prev_loss = loss;
					printf("\r%7.2f%% of max # iterations "
						"(delta = %.5f; stop count = %d)",
						100 * double(t) / double(ITER), max(delta, 0.0),
						STOP_COUNT - stop_count + 1);
					fflush(stdout);
					if (converged)
						break;
				}
	
				// compute max over latent placements
				int M = -1;
				double V = -INFINITY_my;
				//double V = -NWZ;
				for (int m = 0; m < x.num; m++) {
					double val = ex_score(x.seq[m], X, w);
					if (val > V) {
						M = m;
						V = val;
					}
				}
	
				char *ptr = x.seq[M];
				int label = LABEL(ptr);
				if (label * V < 1.0) {
					numupdated++;
					W[i] = 0;
					float *data = EX_DATA(ptr);
					int blocks = NUM_NONZERO(ptr);
					for (int j = 0; j < blocks; j++) {
						int b = BLOCK_IDX(data);
						double mult = (label > 0 ? J : -1) * rateX * X.learnmult[b];
						data++;
						for (int k = 0; k < X.blocksizes[b]; k++)
							w[b][k] += mult * data[k];
						data += X.blocksizes[b];
					}
				}
				else {
					if (W[i] == INCACHE)
						W[i] = MINWAIT + (int)(drand48() * 50);
					//W[i] = MINWAIT + (int)(rand()*50);
					else
						W[i]++;
				}
	
				// periodically regularize the model
				if (t % REGFREQ == 0) {
					// apply lowerbounds
					for (int j = 0; j < X.numblocks; j++)
						for (int k = 0; k < X.blocksizes[j]; k++)
							w[j][k] = max(w[j][k], lb[j][k]);
	
					double rateR = 1.0 / T;
	
	#if FULL_L2 
					// update model
					for (int j = 0; j < X.numblocks; j++) {
						double mult = rateR * X.regmult[j] * X.learnmult[j];
						mult = pow((1 - mult), REGFREQ);
						for (int k = 0; k < X.blocksizes[j]; k++) {
							w[j][k] = mult * w[j][k];
						}
					}
	#else
					// assume simple mixture model
					int maxc = 0;
					double bestval = 0;
					for (int c = 0; c < X.numcomponents; c++) {
						double val = 0;
						for (int i = 0; i < X.componentsizes[c]; i++) {
							int b = X.componentblocks[c][i];
							double blockval = 0;
							for (int k = 0; k < X.blocksizes[b]; k++)
								blockval += w[b][k] * w[b][k] * X.regmult[b];
							val += blockval;
						}
						if (val > bestval) {
							maxc = c;
							bestval = val;
						}
					}
					for (int i = 0; i < X.componentsizes[maxc]; i++) {
						int b = X.componentblocks[maxc][i];
						double mult = rateR * X.regmult[b] * X.learnmult[b];
						mult = pow((1 - mult), REGFREQ);
						for (int k = 0; k < X.blocksizes[b]; k++)
							w[b][k] = mult * w[b][k];
					}
	#endif
				}
			}
		}
	
		if (converged)
			printf("\nTermination criteria reached after %d iterations.\n", t);
		else
			printf("\nMax iteration count reached.\n", t);
	
		free(perm);
		free(W);
		logfile.close();
	}
	
	// score examples
	double *score(data X, char **examples, int num, double **w) {
		double *s = (double *)malloc(sizeof(double)*num);
		check(s != NULL);
		for (int i = 0; i < num; i++)
			s[i] = ex_score(examples[i], X, w);
		return s;
	}
	
	// merge examples with identical labels
	void collapse(data *X, char **examples, int num) {
		collapsed *x = (collapsed *)malloc(sizeof(collapsed)*num);
		check(x != NULL);
		int i = 0;
		x[0].seq = examples;
		x[0].num = 1;
		for (int j = 1; j < num; j++) {
			if (!memcmp(x[i].seq[0] + sizeof(int), examples[j] + sizeof(int),
				labelsize*sizeof(int))) {
				x[i].num++;
			}
			else {
				i++;
				x[i].seq = &(examples[j]);
				x[i].num = 1;
			}
		}
		X->x = x;
		X->num = i + 1;
	}
	
	int main(int argc, char **argv) {
		seed_rand();
		int count;
		data X;
	
		// command line arguments
		check(argc == 12);
		double C = atof(argv[1]);
		double J = atof(argv[2]);
		char *hdrfile = argv[3];
		char *datfile = argv[4];
		char *modfile = argv[5];
		char *inffile = argv[6];
		char *lobfile = argv[7];
		char *cmpfile = argv[8];
		char *objfile = argv[9];
		char *logdir = argv[10];
		char *logtag = argv[11];
	
		// read header file
		FILE *f = fopen(hdrfile, "rb");
		check(f != NULL);
		int header[3];
		count = fread(header, sizeof(int), 3, f);
		check(count == 3);
		int num = header[0];
		labelsize = header[1];
		X.numblocks = header[2];
		X.blocksizes = (int *)malloc(X.numblocks*sizeof(int));
		count = fread(X.blocksizes, sizeof(int), X.numblocks, f);
		check(count == X.numblocks);
		X.regmult = (float *)malloc(sizeof(float)*X.numblocks);
		check(X.regmult != NULL);
		count = fread(X.regmult, sizeof(float), X.numblocks, f);
		check(count == X.numblocks);
		X.learnmult = (float *)malloc(sizeof(float)*X.numblocks);
		check(X.learnmult != NULL);
		count = fread(X.learnmult, sizeof(float), X.numblocks, f);
		check(count == X.numblocks);
		check(num != 0);
		fclose(f);
		printf("%d examples with label size %d and %d blocks\n",
			num, labelsize, X.numblocks);
		printf("block size, regularization multiplier, learning rate multiplier\n");
		dim = 0;
		for (int i = 0; i < X.numblocks; i++) {
			dim += X.blocksizes[i];
			printf("%d, %.2f, %.2f\n", X.blocksizes[i], X.regmult[i], X.learnmult[i]);
		}
	
		// read component info file
		// format: #components {#blocks blk1 ... blk#blocks}^#components
		f = fopen(cmpfile, "rb");
		count = fread(&X.numcomponents, sizeof(int), 1, f);
		check(count == 1);
		printf("the model has %d components\n", X.numcomponents);
		X.componentblocks = (int **)malloc(X.numcomponents*sizeof(int *));
		X.componentsizes = (int *)malloc(X.numcomponents*sizeof(int));
		for (int i = 0; i < X.numcomponents; i++) {
			count = fread(&X.componentsizes[i], sizeof(int), 1, f);
			check(count == 1);
			printf("component %d has %d blocks:", i, X.componentsizes[i]);
			X.componentblocks[i] = (int *)malloc(X.componentsizes[i] * sizeof(int));
			count = fread(X.componentblocks[i], sizeof(int), X.componentsizes[i], f);
			check(count == X.componentsizes[i]);
			for (int j = 0; j < X.componentsizes[i]; j++)
				printf(" %d", X.componentblocks[i][j]);
			printf("\n");
		}
		fclose(f);
	
		// read examples
		f = fopen(datfile, "rb");
		check(f != NULL);
		printf("Reading examples\n");
		char **examples = (char **)malloc(num*sizeof(char *));
		check(examples != NULL);
		for (int i = 0; i < num; i++) {
			// we use an extra byte in the end of each example to mark unique
			// we use an extra int at the start of each example to store the 
			// example's byte length (excluding unique flag and this int)
			//int buf[labelsize+2];
			int *buf = new int[labelsize + 2];
	
			count = fread(buf, sizeof(int), labelsize + 2, f);
			check(count == labelsize + 2);
			// byte length of an example's data segment
			int len = sizeof(int)*(labelsize + 2) + sizeof(float)*buf[labelsize + 1];
			// memory for data, an initial integer, and a final byte
			examples[i] = (char *)malloc(sizeof(int) + len + 1);
			check(examples[i] != NULL);
			// set data segment's byte length
			((int *)examples[i])[0] = len;
			// set the unique flag to zero
			examples[i][sizeof(int) + len] = 0;
			// copy label data into example
			for (int j = 0; j < labelsize + 2; j++)
				((int *)examples[i])[j + 1] = buf[j];
			// read the rest of the data segment into the example
			count = fread(examples[i] + sizeof(int)*(labelsize + 3), 1,
				len - sizeof(int)*(labelsize + 2), f);
			check(count == len - sizeof(int)*(labelsize + 2));
	
			delete[] buf;
		}
		fclose(f);
		printf("done\n");
	
		// sort
		printf("Sorting examples\n");
		char **sorted = (char **)malloc(num*sizeof(char *));
		check(sorted != NULL);
		memcpy(sorted, examples, num*sizeof(char *));
		qsort(sorted, num, sizeof(char *), comp);
		printf("done\n");
	
		// find unique examples
		int i = 0;
		int len = *((int *)sorted[0]);
		sorted[0][sizeof(int) + len] = 1;
		for (int j = 1; j < num; j++) {
			int alen = *((int *)sorted[i]);
			int blen = *((int *)sorted[j]);
			if (alen != blen ||
				memcmp(sorted[i] + sizeof(int), sorted[j] + sizeof(int), alen)) {
				i++;
				sorted[i] = sorted[j];
				sorted[i][sizeof(int) + blen] = 1;
			}
		}
		int num_unique = i + 1;
		printf("%d unique examples\n", num_unique);
	
		// collapse examples
		collapse(&X, sorted, num_unique);
		printf("%d collapsed examples\n", X.num);
	
		// initial model
		double **w = (double **)malloc(sizeof(double *)*X.numblocks);
		check(w != NULL);
		f = fopen(modfile, "rb");
		for (int i = 0; i < X.numblocks; i++) {
			w[i] = (double *)malloc(sizeof(double)*X.blocksizes[i]);
			check(w[i] != NULL);
			count = fread(w[i], sizeof(double), X.blocksizes[i], f);
			check(count == X.blocksizes[i]);
		}
		fclose(f);
	
		// lower bounds
		double **lb = (double **)malloc(sizeof(double *)*X.numblocks);
		check(lb != NULL);
		f = fopen(lobfile, "rb");
		for (int i = 0; i < X.numblocks; i++) {
			lb[i] = (double *)malloc(sizeof(double)*X.blocksizes[i]);
			check(lb[i] != NULL);
			count = fread(lb[i], sizeof(double), X.blocksizes[i], f);
			check(count == X.blocksizes[i]);
		}
		fclose(f);
	
		// train
		printf("Training\n");
		gd(C, J, X, w, lb, logdir, logtag);
		printf("done\n");
	
		// save model
		printf("Saving model\n");
		f = fopen(modfile, "wb");
		check(f != NULL);
		for (int i = 0; i < X.numblocks; i++) {
			count = fwrite(w[i], sizeof(double), X.blocksizes[i], f);
			check(count == X.blocksizes[i]);
		}
		fclose(f);
	
		// score examples
		printf("Scoring\n");
		double *s = score(X, examples, num, w);
	
		// Write info file
		printf("Writing info file\n");
		f = fopen(inffile, "w");
		check(f != NULL);
		for (int i = 0; i < num; i++) {
			int len = ((int *)examples[i])[0];
			// label, score, unique flag
			count = fprintf(f, "%d\t%f\t%d\n", ((int *)examples[i])[1], s[i],
				(int)examples[i][sizeof(int) + len]);
			check(count > 0);
		}
		fclose(f);
	
		// compute loss and write it to a file
		double lossinfo[3];
		compute_loss(lossinfo, C, J, X, w);
		printf("Writing objective function info file\n");
		f = fopen(objfile, "w");
		count = fprintf(f, "%f\t%f\t%f", lossinfo[0], lossinfo[1], lossinfo[2]);
		check(count > 0);
		fclose(f);
	
		printf("Freeing memory\n");
		for (int i = 0; i < X.numblocks; i++) {
			free(w[i]);
			free(lb[i]);
		}
		free(w);
		free(lb);
		free(s);
		for (int i = 0; i < num; i++)
			free(examples[i]);
		free(examples);
		free(sorted);
		free(X.x);
		free(X.blocksizes);
		free(X.regmult);
		free(X.learnmult);
		for (int i = 0; i < X.numcomponents; i++)
			free(X.componentblocks[i]);
		free(X.componentblocks);
		free(X.componentsizes);
	
		return 0;
	}

   - 下载svm_mex601工具包到voc-release4.01文件夹中

   - 修改global.m配置路径，注释掉判断文件夹是否存在的代码，配置结果路径，暂存数据路径和PASCAL VOC development kit and dataset

### LINK
1、[http://blog.csdn.net/dreamd1987/article/details/7399151](http://blog.csdn.net/dreamd1987/article/details/7399151 "train 移植1") train 移植1

2、[http://blog.csdn.net/pozen/article/details/7103412](http://blog.csdn.net/pozen/article/details/7103412 "train 移植2") train 移植2

3、[http://blog.sina.com.cn/s/blog_4af4d81f0101dk3u.html](http://blog.sina.com.cn/s/blog_4af4d81f0101dk3u.html "train 移植3") train 移植3

4、[http://blog.sina.com.cn/s/blog_4af4d81f0101dk38.html](http://blog.sina.com.cn/s/blog_4af4d81f0101dk38.html "detect 移植1") detect 移植1

5、[http://www.tuicool.com/articles/UreMvm](http://www.tuicool.com/articles/UreMvm "完整移植") 完整移植

6、[https://sourceforge.net/projects/mex-svm/files/Source/](https://sourceforge.net/projects/mex-svm/files/Source/ "svm_mex601 工具包") svm_mex601 工具包下载

7、[http://host.robots.ox.ac.uk/pascal/VOC/voc2007/index.html](http://host.robots.ox.ac.uk/pascal/VOC/voc2007/index.html "PASCAL VOC2007") PASCAL VOC2007 数据集和工具包

8、[http://people.eecs.berkeley.edu/~rbg/latent/index.html](http://people.eecs.berkeley.edu/~rbg/latent/index.html "DPM source code")  DPM source code
