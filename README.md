import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.Rect;
import android.view.MotionEvent;
import android.view.SurfaceHolder;
import android.view.SurfaceView;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

// מחלקה לאובייקט משחק
class GameObject {
    private Bitmap bitmap; // תמונה של האובייקט
    private int x, y; // מיקום על המסך
    private boolean isSelected = false; // האם האובייקט נבחר

    public GameObject(Bitmap bitmap, int x, int y) {
        this.bitmap = bitmap;
        this.x = x;
        this.y = y;
    }

    public void draw(Canvas canvas) {
        if (isSelected) {
            // הדגשה אם האובייקט נבחר
            Paint paint = new Paint();
            paint.setColor(Color.RED);
            paint.setStrokeWidth(5);
            paint.setStyle(Paint.Style.STROKE);
            canvas.drawRect(getBounds(), paint);
        }
        canvas.drawBitmap(bitmap, x, y, null);
    }

    public Rect getBounds() {
        return new Rect(x, y, x + bitmap.getWidth(), y + bitmap.getHeight());
    }

    public boolean isTouched(int touchX, int touchY) {
        return getBounds().contains(touchX, touchY);
    }

    public void select() {
        isSelected = true;
    }

    public void deselect() {
        isSelected = false;
    }
}

// מחלקה לניהול אירועים
class EventManager {
    private Map<GameObject, List<EventListener>> eventListeners;

    public EventManager() {
        eventListeners = new HashMap<>();
    }

    public void addEventListener(GameObject gameObject, EventListener listener) {
        if (!eventListeners.containsKey(gameObject)) {
            eventListeners.put(gameObject, new ArrayList<>());
        }
        eventListeners.get(gameObject).add(listener);
    }

    public void handleTouch(int touchX, int touchY) {
        for (GameObject obj : eventListeners.keySet()) {
            if (obj.isTouched(touchX, touchY)) {
                List<EventListener> listeners = eventListeners.get(obj);
                for (EventListener listener : listeners) {
                    listener.onEvent();
                }
            }
        }
    }
}

// ממשק לאירועים
interface EventListener {
    void onEvent();
}

// אובייקט פיזיקה מתקדם
class PhysicsObject extends GameObject {
    private float velocityX, velocityY;
    private float gravity = 0.5f;

    public PhysicsObject(Bitmap bitmap, int x, int y) {
        super(bitmap, x, y);
        this.velocityX = 0;
        this.velocityY = 0;
    }

    @Override
    public void update() {
        velocityY += gravity;
        setPosition((int) (getBounds().left + velocityX), (int) (getBounds().top + velocityY));
    }

    public void setVelocity(float velocityX, float velocityY) {
        this.velocityX = velocityX;
        this.velocityY = velocityY;
    }
}

// עורך המשחק
class GameEditorView extends SurfaceView {
    private List<GameObject> gameObjects = new ArrayList<>();
    private EventManager eventManager;
    private SurfaceHolder surfaceHolder;

    public GameEditorView(Context context) {
        super(context);
        surfaceHolder = getHolder();
        eventManager = new EventManager();
    }

    public void addGameObject(GameObject gameObject) {
        gameObjects.add(gameObject);
        eventManager.addEventListener(gameObject, new EventListener() {
            @Override
            public void onEvent() {
                // מה לעשות כשאובייקט נוגע
                gameObject.select();
            }
        });
    }

    @Override
    public void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.drawColor(Color.WHITE); // צבע רקע
        for (GameObject gameObject : gameObjects) {
            gameObject.draw(canvas);
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int touchX = (int) event.getX();
        int touchY = (int) event.getY();

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                eventManager.handleTouch(touchX, touchY);
                invalidate(); // רענן את המסך
                return true;
            case MotionEvent.ACTION_MOVE:
                // אפשר להוסיף לוגיקה להעברת אובייקטים
                break;
            case MotionEvent.ACTION_UP:
                break;
        }
        return super.onTouchEvent(event);
    }
}

// MainActivity - הכיתה הראשית של האפליקציה
public class MainActivity extends AppCompatActivity {
    private GameEditorView gameEditorView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        gameEditorView = new GameEditorView(this);
        setContentView(gameEditorView);

        // הוספת אובייקטים לדוגמה
        Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.example_image);
        GameObject gameObject = new GameObject(bitmap, 100, 100);
        gameEditorView.addGameObject(gameObject);
    }
}
