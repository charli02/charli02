### Hi there 👋

**charli02/charli02** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
public abstract class RepositoryBase<TEntity, TContext> :
        IRepository<TEntity> where TEntity : class
                                where TContext : DbContext, new()
    {

        private TContext _context;// = new TContext();
        public TContext Context
        {
            get { return _context; }
            set { _context = value; }
        }

        // not needed, just keeping it, will delete later
        protected DbSet<TEntity> _DbSet { get; set; }

        public RepositoryBase()
        {
        }

        public RepositoryBase(TContext context)
        {
            _context = context;
            _DbSet = _context.Set<TEntity>();
        }

        //public List<TEntity> GetAll()
        //{
        //    if (_DbSet.Count() > 0)
        //        return _DbSet.ToList();

        //    return new List<TEntity>();
        //}

        public IEnumerable<TEntity> GetAll()
        {
            IEnumerable<TEntity> query = _context.Set<TEntity>();
            return query;
        }

        public IEnumerable<TEntity> FindBy(Expression<Func<TEntity, bool>> predicate)
        {
            IEnumerable<TEntity> query = _context.Set<TEntity>().Where(predicate);
            return query;
        }

        public TEntity Get(Guid id)
        {
            return _DbSet.Find(id);
        }

        //3. Mark entity as modified
        public void MarkModified(TEntity entity)
        {
            _context.Entry<TEntity>(entity).State = EntityState.Modified;
        }

        public void Add(TEntity entity)
        {
            _context.Set<TEntity>().Add(entity);
            //_DbSet.Add(entity);
        }

        // Update/Save Entity in database
        public void Edit(TEntity entity)
        {
            _context.Entry<TEntity>(entity).State = EntityState.Modified;
        }

        public void Delete(int id)
        {
            var entity = _context.Set<TEntity>().Find(id);
            if (entity != null)
                _context.Set<TEntity>().Remove(entity);

            //_DbSet.Remove(entity);
        }

        public void Delete(TEntity entity)
        {
            _context.Set<TEntity>().Remove(entity);
            //_DbSet.Remove(entity);
        }

        public void Save()
        {
            _context.SaveChanges();
        }

        public void AddRange(IEnumerable<TEntity> entities)
        {
            _context.Set<TEntity>().AddRange(entities);
        }

        public void DeleteRange(IEnumerable<TEntity> entities)
        {
            _context.Set<TEntity>().RemoveRange(entities);
        }

        public void EditRange(IEnumerable<TEntity> entities)
        {
            foreach (TEntity entity in entities)
                _context.Entry<TEntity>(entity).State = EntityState.Modified;
        }

        public int GetCount()
        {
            return _DbSet.Count();
        }

        public int GetCount(Expression<Func<TEntity, bool>> predicate)
        {
            return _DbSet.Count(predicate);
        }

        //public virtual List<TEntity> GetWithRawSql(string query, params object[] parameters)
        //{
        //    return _DbSet.SqlQuery(query, parameters).ToList();
        //}
    }
    
    ------------------------------------------------------------------------------------------------------
    
    public interface IRepository<TEntity> where TEntity : class
    {
        TEntity Get(Guid id);
        IEnumerable<TEntity> GetAll();
        IEnumerable<TEntity> FindBy(Expression<Func<TEntity, bool>> predicate);

        int GetCount();
        int GetCount(Expression<Func<TEntity, bool>> predicate);

        void MarkModified(TEntity entity);
        void Add(TEntity entity);
        void AddRange(IEnumerable<TEntity> entities);

        void Delete(int id);
        void Delete(TEntity entity);
        void DeleteRange(IEnumerable<TEntity> entities);

        void Edit(TEntity entity);
        void EditRange(IEnumerable<TEntity> entities);

        void Save();

        // List<TEntity> GetWithRawSql(string query, params object[] parameters);
    }
    
    
    ------------------------------------------------------------------------------------------------------
    public class DesignationHierarchyRepository : RepositoryBase<DesignationHierarchy, XLMSDbContext>, IDesignationHierarchyRepository
    {
        //private XLMSDbContext _context;

        public DesignationHierarchyRepository(XLMSDbContext context) : base(context)
        {
            //_context = context;
        }

    }

------------------------------------------------------------------------------------------------------

public interface IDesignationHierarchyRepository : IRepository<DesignationHierarchy>
    {
    }
 ------------------------------------------------------------------------------------------------------
 public class DesignationHierarchyService : IDesignationHierarchyService
    {
        private readonly IDesignationHierarchyRepository _designationHierarchyRepository;
        private readonly IDesignationService _designationService;

        public DesignationHierarchyService(IDesignationHierarchyRepository designationHierarchyRepository, IDesignationService designationService)
        {
            _designationHierarchyRepository = designationHierarchyRepository;
            _designationService = designationService;
        }
        public bool ResetDesignationHierarchies(IList<DesignationHierarchy> designationHierarchies)
        {
            var oldDesignationHierarchies = _designationHierarchyRepository.GetAll().ToList();
            _designationHierarchyRepository.DeleteRange(oldDesignationHierarchies);
            _designationHierarchyRepository.AddRange(designationHierarchies);
            _designationHierarchyRepository.Save();
            return true;
        }

       

    }
 ------------------------------------------------------------------------------------------------------
 public class PagedResponse<T> : ApiResponse<T>
    {
        public int PageNumber { get; set; }
        public int PageSize { get; set; }

        public PagedResponse(IEnumerable<T> data, int pageNumber, int pageSize, string message, bool succeeded, Enums.HttpStatusCode responseCode)
        {
            this.PageNumber = pageNumber;
            this.PageSize = pageSize;
            this.Data = data;
            this.Message = message;
            this.Succeeded = succeeded;
            this.ResponseCode = responseCode;
        }

        public static PagedResponse<T> PagedList(IEnumerable<T> source, string message, bool succeeded, Enums.HttpStatusCode responseCode, int pageNumber = 0, int pageSize = 0)
        {
            if (pageNumber > 0 && pageSize > 0)
            {
                var data = source.Skip((pageNumber - 1) * pageSize).Take(pageSize);
                return new PagedResponse<T>(data, pageNumber, pageSize, message, succeeded, responseCode);
            }

            return new PagedResponse<T>(source, pageNumber, pageSize, message, succeeded, responseCode);
        }
    }
    
    ------------------------------------------------------------------------------------------------------
    public class PaginationFilter
    {
        public int PageNumber { get; set; }
        public int PageSize { get; set; }
        public int GetActiveUsrs { get; set; }
        public string QuerySearch {get; set;}
        public int GetActiveGroups { get; set; }
        public int GetInActiveGroups { get; set; }
        public int GetActiveLocations { get; set; }
        //public int GetIsDeleted { get; set; }
        
        public PaginationFilter()
        {
            this.PageNumber = 1;
            this.PageSize = 5;
            this.QuerySearch = "";
            this.GetActiveUsrs = 1;
        }
        public PaginationFilter(int pageNumber, int pageSize)
        {
            this.PageNumber = pageNumber < 1 ? 1 : pageNumber;
            this.PageSize = pageSize > 5 ? 5 : pageSize;
        }

        //public PaginationFilter(int pageNumber, int pageSize, string QuerySearch)
        //{
        //    this.PageNumber = pageNumber < 1 ? 1 : pageNumber;
        //    this.PageSize = pageSize > 5 ? 5 : pageSize;
        //    this.QuerySearch = string.IsNullOrEmpty(QuerySearch) == true ? "" : QuerySearch;
        //}

        public PaginationFilter(int pageNumber, int pageSize, int GetActiveUsrs ,string QuerySearch)
        {
            this.PageNumber = pageNumber < 1 ? 1 : pageNumber;
            this.PageSize = pageSize > 5 ? 5 : pageSize;
            this.GetActiveUsrs = GetActiveUsrs == 1 ? 1 : GetActiveUsrs;
            this.QuerySearch = string.IsNullOrEmpty(QuerySearch) == true ? "" : QuerySearch;
            
        }
    }
 ------------------------------------------------------------------------------------------------------
 [HttpGet]
        [Route(Constants.Attrributes.ListApiName)]
        public PagedResponse<ResultUser> GetUsers([FromQuery] PaginationFilter filter)
        {
            //ELIMINATE USERS WHOSE PROFILE ARE NULL 
            IEnumerable<ResultUser> users = _userService.GetUsers().Where(x => x.UserProfile != null).OrderByDescending(x => x.UserProfile.ModifiedOn != null ? x.UserProfile.ModifiedOn : x.UserProfile.CreatedOn);

            try
            {
                if (users != null && users.Count() > 0)
                {
                    //CHECK FOR ACTIVE AND ALL USERS BESED ON PAGINATION PARAMETER
                    if (filter.GetActiveUsrs == 1)
                        users = users.Where(x => x.IsActive);
                    if (filter.GetActiveUsrs == 2)
                        users = users.Where(x => !(x.IsActive));

                    //FUNCTION FOR SEARCH USER BY FULLNAME OR EMAIL
                    //Search Parameter [With null check]  
                    if (!string.IsNullOrEmpty(filter.QuerySearch))
                    {
                        filter.QuerySearch = filter.QuerySearch.ToLower();
                        users = users.Where(a => (a.UserName.ToLower().Contains(filter.QuerySearch) || String.Format("{0} {1}", a.UserProfile == null ? "" : a.UserProfile.FirstName, a.UserProfile == null ? "" : a.UserProfile.LastName).ToLower().Contains(filter.QuerySearch)));

                    }
                    response = Constants.ApiRequestResponse.ResponseSuccess;
                    _commonService.ApiRequestLogInDb(Request.Path.Value, response, getUserEmail(), CommonFunction.returnJsontoString(filter), ErrMsg);
                    return PagedResponse<ResultUser>.PagedList(
                        users,
                        Constants.Messages.Success,
                        true,
                        Enums.HttpStatusCode.OK,
                        filter.PageNumber,
                        filter.PageSize);
                }
                else
                {
                    ErrMsg = Constants.Messages.NotDataExistsInTable;
                    _commonService.ApiRequestLogInDb(Request.Path.Value, response, getUserEmail(), CommonFunction.returnJsontoString(filter), ErrMsg);
                    return PagedResponse<ResultUser>.PagedList(
                        users,
                        Constants.Messages.NotDataExistsInTable,
                        true,
                        Enums.HttpStatusCode.OK,
                        filter.PageNumber,
                        filter.PageSize);
                }
            }
            catch (Exception ex)
            {
                return PagedResponse<ResultUser>.PagedList(
                    users,
                    Constants.Messages.Failed,
                    false,
                    Enums.HttpStatusCode.InternalServerError,
                    filter.PageNumber,
                    filter.PageSize);
            }
        }
