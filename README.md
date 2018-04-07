# OMR_Scanner_Tutorial_android_openCV
This is a simple short tutorial on how to use Opencv in java to scan OMR sheets and interpret answers. I have developed a solution based on this so this is just a snippet of that project. Since oppencv java doest not  have much community support i hope this helps in your java or android projects. 

## Procedure

# Step 1 : Load opencv into your android project 
 follow this link for tutorial  on that
 https://docs.opencv.org/2.4/doc/tutorials/introduction/android_binary_package/dev_with_OCV_on_Android.html
 
 
 #  Step 2 :
 Imports 
 import org.opencv.android.BaseLoaderCallback;
import org.opencv.android.LoaderCallbackInterface;
import org.opencv.android.Utils;
import org.opencv.core.Core;
import org.opencv.core.CvType;
import org.opencv.core.Mat;
import org.opencv.core.MatOfPoint;
import org.opencv.core.MatOfPoint2f;
import org.opencv.core.MatOfRect;
import org.opencv.core.Point;
import org.opencv.core.Rect;
import org.opencv.core.Scalar;
import org.opencv.core.Size;
import org.opencv.imgproc.Imgproc;
import org.opencv.android.OpenCVLoader;
import org.opencv.utils.Converters;

import static android.content.Context.MODE_PRIVATE;
import static org.opencv.core.Core.bitwise_and;
import static org.opencv.core.Core.bitwise_not;
import static org.opencv.core.Core.countNonZero;
import static org.opencv.core.Core.sort;
import static org.opencv.core.Core.subtract;
import static org.opencv.core.CvType.CV_64FC1;
import static org.opencv.core.CvType.CV_8UC1;
import static org.opencv.core.CvType.CV_8UC4;
import static org.opencv.imgproc.Imgproc.COLOR_RGB2GRAY;
import static org.opencv.imgproc.Imgproc.CV_DIST_HUBER;
import static org.opencv.imgproc.Imgproc.CV_HOUGH_STANDARD;
import static org.opencv.imgproc.Imgproc.boundingRect;
import static org.opencv.imgproc.Imgproc.contourArea;


# Step 3: Opencv module loading
// private BaseLoaderCallback mLoaderCallback = new BaseLoaderCallback(this) {
//        @Override
//        public void onManagerConnected(int status) {
//            switch (status) {
//                case LoaderCallbackInterface.SUCCESS:
//                {
//                    Log.i("opencv", "OpenCV loaded successfully");
//                    localMat = new Mat();
//                    image=BitmapFactory.decodeFile(fileUri.getPath());
//                    //localMat = borderDetect(image);
//                    //Utils.matToBitmap(localMat,image);
//                } break;
//                default:
//                {
//                    super.onManagerConnected(status);
//                } break;
//            }
//        }
//    };

//    @Override
//    public void onResume()
//    {
//        super.onResume();
//        OpenCVLoader.initAsync(OpenCVLoader.OPENCV_VERSION_2_4_6, this, mLoaderCallback);
//    }

# Step 4 : take the image file
 filename="/storage/emulated/0/OMR_Sheets/Test.jpg";
 fileUri = Uri.parse(filename);
 
# Step 5 : convert image ti bitmap
  BitmapFactory.Options options = new BitmapFactory.Options();
  image=BitmapFactory.decodeFile(fileUri.getPath());
  
 # Step 6 :  follow this method
 
    public void showAllCircles(Bitmap paramView)
    {
        //paramView = BitmapFactory.decodeFile(fileUri.getPath());
        Mat localMat1 = new Mat();
        Utils.bitmapToMat(paramView, localMat1);

        Mat localMat2 = new Mat();
        Imgproc.GaussianBlur(localMat1, localMat2, new Size(5.0D, 5.0D), 7.0D, 6.5D);
        Object localObject = new Mat();
        Imgproc.cvtColor(localMat2, (Mat)localObject, COLOR_RGB2GRAY);
        Mat cloneMat= ((Mat) localObject).clone();
        //Mat blackwhite= ((Mat) localObject).clone();
        localMat2 = localMat1.clone();
        bitwise_not(cloneMat,cloneMat);
        Imgproc.threshold(cloneMat,localMat2,127,255,Imgproc.THRESH_OTSU);
        Mat thresh=localMat2.clone();

        List<MatOfPoint> contours = new ArrayList<MatOfPoint>();
        List<MatOfPoint> questions = new ArrayList<MatOfPoint>();
        List<MatOfPoint> sorted = new ArrayList<MatOfPoint>();

        Mat hierarchy = new Mat();
        Imgproc.findContours(localMat2, contours, hierarchy, Imgproc.RETR_EXTERNAL, Imgproc.CHAIN_APPROX_SIMPLE);
        Rect rect,rect2;
        int groups[] = new int[30];
        int i=0,l=0;
        for (int contourIdx = 0; contourIdx < contours.size(); contourIdx++) {
            rect = Imgproc.boundingRect(contours.get(contourIdx));
            //float area=rect.width/(float)rect.height;


            if(rect.width>=30 && rect.height>=30){
                questions.add(contours.get(contourIdx));

                //rect3 = Imgproc.boundingRect(questions.get(i));
                //Log.i("Before------",rect3.tl()+" ");

                for(int ctr=0;ctr<questions.size()-1;ctr++){
                    MatOfPoint ctr1 = questions.get(i);
                    rect = Imgproc.boundingRect(questions.get(i));
                    MatOfPoint ctr2 = questions.get(ctr);
                    rect2 = Imgproc.boundingRect(questions.get(ctr));
                    if(rect.tl().x<rect2.tl().x){
                        questions.set(ctr,ctr1);
                        questions.set(i,ctr2);
                    }
                }
                //rect3 = Imgproc.boundingRect(questions.get(i));
                Log.i("after",rect3.tl()+" ");
                i++;

            }
        }

        int j=0;i=0;
        while(j!=questions.size()){

            for(int ctr=0;ctr<questions.size()-1;ctr++){
                MatOfPoint ctr1 = questions.get(i);
              rect = Imgproc.boundingRect(questions.get(i));
                MatOfPoint ctr2 = questions.get(ctr);
                rect2 = Imgproc.boundingRect(questions.get(ctr));
                if(rect.tl().y<rect2.tl().y){
                    questions.set(ctr,ctr1);
                    questions.set(i,ctr2);
                }
            }
        i++;
       j++;
        }

        Collections.sort(questions, new Comparator<MatOfPoint>() {

            @Override
            public int compare(MatOfPoint lhs, MatOfPoint rhs) {
                return Double.valueOf(boundingRect(lhs).tl().y).compareTo(boundingRect(rhs).tl().y);
            }
        });

        List<MatOfPoint> questionSortedL = new ArrayList<>();

        List<MatOfPoint> questionSortedR = new ArrayList<>();

        List<MatOfPoint> sheetid = new ArrayList<>();

        List<MatOfPoint> questionSort = new ArrayList<MatOfPoint>();

        for (int index=0; index<=7; index++){
            MatOfPoint questionsort = questions.get(index);
            sheetid.add(questionsort);
        }

        Collections.sort(sheetid, new Comparator<MatOfPoint>() {

            @Override
            public int compare(MatOfPoint lhs, MatOfPoint rhs) {
                return Double.valueOf(boundingRect(lhs).tl().x).compareTo(boundingRect(rhs).tl().x);
            }
        });

        double decimal=0,binaryPlace=7;
        for (int contourIdx = 0; contourIdx < sheetid.size(); contourIdx++) {

            Rect rectCrop = boundingRect(sheetid.get(contourIdx));
            Mat imageROI= thresh.submat(rectCrop);

            int total = countNonZero(imageROI);
            double pixel =total/contourArea(sheetid.get(contourIdx))*100;
            if(pixel>=100 && pixel<=130){

                //s=s+(count+1)+"->"+((contourIdx%5)+1)+" , ";
                //Log.i("Answer:",(count+1)+" - "+((contourIdx%5)+1));
                //answers.add((contourIdx%5)+1);
                //Imgproc.drawContours(localMat1, questionSortedL, contourIdx, new Scalar(0,255,0), 10);
                decimal+=Math.pow(2,binaryPlace);
            }
            binaryPlace--;
        }

        int intDecimal=(int)decimal;
        sheetId=String.valueOf(intDecimal).trim();
        int start=0,end=0,leftIdx=0,rightIdx=0;

        for (int temp=0;temp<questions.size()/10;temp++) {

            end=end+10;

            for (int questionidx = start; questionidx < end; questionidx++) {
                MatOfPoint questionsort = questions.get(questionidx);
                questionSort.add(questionidx-start, questionsort);
            }

            Collections.sort(questionSort, new Comparator<MatOfPoint>() {

                @Override
                public int compare(MatOfPoint lhs, MatOfPoint rhs) {
                    return Double.valueOf(boundingRect(lhs).tl().x).compareTo(boundingRect(rhs).tl().x);
                }
            });

            for (int questionidx = start; questionidx < end; questionidx++) {
                MatOfPoint questionsort = questionSort.get(questionidx-start);
                if (questionidx < end-5) {

                    questionSortedL.add(leftIdx, questionsort);
                    leftIdx++;
                } else {
                    questionSortedR.add(rightIdx, questionsort);
                    rightIdx++;
                }
            }

            start=start+10;
            questionSort.clear();
        }

        int count=0;

        String s="Answers: ";



        for (int contourIdx = 0; contourIdx < questionSortedL.size(); contourIdx++) {

            Rect rectCrop = boundingRect(questionSortedL.get(contourIdx));
            Mat imageROI= thresh.submat(rectCrop);

            int total = countNonZero(imageROI);
            double pixel =total/contourArea(questionSortedL.get(contourIdx))*100;
            if(pixel>=100 && pixel<=130){
                Log.d("pixel-area",""+pixel);
                s=s+(count+1)+"->"+((contourIdx%5)+1)+" , ";
                //Log.i("Answer:",(count+1)+" - "+((contourIdx%5)+1));
                answers.add((contourIdx%5)+1);
                Imgproc.drawContours(localMat1, questionSortedL, contourIdx, new Scalar(0,255,0), 10);
                count++;
            }

        }

        for (int contourIdx = 0; contourIdx < questionSortedR.size(); contourIdx++) {

            Rect rectCrop = boundingRect(questionSortedR.get(contourIdx));
            Mat imageROI= thresh.submat(rectCrop);

            int total = countNonZero(imageROI);
            double pixel =total/contourArea(questionSortedR.get(contourIdx))*100;
            if(pixel>=100 && pixel<=130){

                s=s+(count+1)+"->"+((contourIdx%5)+1)+" , ";
                //Log.i("Answer:",(count+1)+" - "+(contourIdx%5+1));
                answers.add((contourIdx%5)+1);
                Imgproc.drawContours(localMat1, questionSortedR, contourIdx, new Scalar(0,255,0), 10);
                count++;
            }

        }

        int questionsize=questionsArrayList.size();
        Log.d("questionssize  : ",""+questionsize);
        answers2= new ArrayList<Integer> (answers.subList(0,questionsize));

        Utils.matToBitmap(localMat1, paramView);
        this.imageView.setImageBitmap(paramView);

    }
