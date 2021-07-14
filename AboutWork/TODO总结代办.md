public class SelectSpinner extends Spinner {
    OnItemSelectedListener listener;

    public SelectSpinner(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public void setSelection(int position) {
        super.setSelection(position);
        if (listener != null)
            listener.onItemSelected(null, null, position, 0);
    }

    public void setOnItemSelectedChangedListener(
            OnItemSelectedListener listener) {
        this.listener = listener;
    }
}

Spinner 在ListView此类容器中的问题与解决方案

AndroidSwitch 在ListView此类容器中的问题与解决方案

不响应setOnClickListener
